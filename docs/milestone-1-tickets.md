# Milestone 1 tickets

The ordered work for the first combat milestone: **Basic Attack → Dash → Tidal Bolt → Arc Burst → Wet Reaction**. The design lives in the [first combat prototype](first-combat-prototype.md), the [code structure](code-structure.md) and the [playtest plan](playtest-plan.md).

Work stops after each ticket until the developer gives the go-ahead. Game feel is judged by playing, so each ticket lists what only a person in Studio can check.

## Checkpoints

| Date | Checkpoint | Contents |
|---|---|---|
| 2026-11-06 | `Riftfall Arena.rbxl` at git commit `806f524` (saved, not published) | Empty baseplate, spawn and default lighting, with no scripts |
| 2026-11-06 | Published place `134091394433436`, version 2 | Ticket 1: the skeleton, `Config`, `Rules`, `Protocol` and tests |
| 2026-11-06 | Published place `134091394433436`, version 9 (private) | Tickets 2 and 3: the test space and the Basic Attack against a static dummy |
| 2026-11-07 | No publish of its own, first published in version 20 | Ticket 4: Dash, Evade, Movement Validation and the moving dummies |
| 2026-11-07 | No publish of its own, first published in version 20 | Ticket 5: Tidal Bolt, Wet, Knockback and the Ability input and cooldown display |
| 2026-11-07 | Published place `134091394433436`, version 20 (private) | Ticket 6: Arc Burst, the shared Reaction system and the Wet Reaction's payoff feedback |
| 2026-11-07 | Published place `134091394433436`, version 24 (private) | Ticket 7: health, Natural Recovery, Kill Recovery, death and respawn, kill and low-health feedback, session logs and the shooting dummy |
| 2026-11-07 | Published place `134091394433436`, version 26 (private) | Ticket 8's Stage 2 fixes: the prototype spawn rule, and kill notices naming test players |

Roblox version history for the published place is the code's history, as set out in [ADR-0005](adr/0005-game-code-lives-in-studio-edited-through-mcp.md). Code edits only reach it when Studio publishes, so publish at the end of each accepted ticket. The `.rbxl` in this repo is a snapshot from the same point, and it goes stale between checkpoints.

Autosaves also take version numbers, so a version only counts as a checkpoint if Version History marks it Published. On 2026-11-07 that was versions 2, 9, 20, 24 and 26, and the earlier numbers recorded here for Tickets 4–7 (10, 11, 12 and 13) were autosaves from 2026-11-06. Tickets were matched to publishes by comparing Version History's times with the commit times, without opening the versions' contents.

## 1. Foundation

**Scope:** the folder skeleton from the code structure, `Config` for the first loop's Basic Attack, Abilities, Status and Reaction, pure `ReactionRules` and `RankRules`, `Protocol` with payload validation, and small tests.

**Acceptance checks:**

- The layout in the Explorer matches the code structure, plus the `ServerStorage/Tests` addition.
- Tests pass for Wet and Lightning Reaction resolution, Status consumption and the other setup outcomes, Reaction ownership and credit, self-interaction, and Rank progression.
- Tests show that the Basic Attack triggers no Reaction, that Tidal Bolt into Arc Burst lands at 40–45% of health, and that it stays within the burst limits at every Rank.
- Malformed `Protocol` payloads are rejected, including NaN and infinite values.
- Pressing Play gives both Mains a clean start, with every RemoteEvent visible on the client and no console errors.

**Test in Studio:** press Play, confirm the Output has no errors and shows both startup lines, then stop. Nothing is playable yet.

## 2. Test space

**Scope:** the greybox arena (about 100 studs across, with cover, one raised area and one ledge or drop), the Prototype Character with a player accent and name label, the fixed Hitbox, the over-the-shoulder camera, the Crosshair and Aim Point. Turn off `StreamingEnabled`, which is on by default and could hide a distant character from a client during combat.

**Acceptance checks:**

- Every player spawns as the same Prototype Character, whatever their avatar is.
- The Hitbox is the same size for all players and does not follow the visible model's size.
- The Crosshair stays at the screen's centre, and the Aim Point resolves to the world under it, ignoring the player's own character.
- The camera does not clip badly through cover.

**Test in Studio:** walk the arena. Judge camera shoulder offset, distance and sensitivity, whether the Crosshair reads clearly, whether the arena's scale and cover feel right, and whether the character's silhouette is readable at 60 studs.

## 3. Basic Attack

**Scope:** the full path against a static dummy: hold-to-fire input, `CastRequest`, server validation, Authoritative Projectile, `CastStarted`, Cosmetic Projectile, Rejected Cast, Hit Tolerance, Server Confirmation, `CombatResult`, and attacker and victim hit feedback.

**Acceptance checks:**

- The server caps cadence at about 0.4 s, even if a client sends requests faster.
- Damage reaches the dummy only from the Authoritative Projectile, and projectiles disappear harmlessly at about 60 studs.
- A forged or invalid Cast is rejected and the caster's client cancels its visuals.
- Hit marker, sound and damage number appear only after Server Confirmation.
- One `CombatResult` drives all the hit feedback, with no extra RemoteEvents.

**Test in Studio:** hold fire at the dummy from several ranges while moving. Judge projectile speed, firing responsiveness, whether hits feel fair, and how the hit marker and sound feel, also with simulated latency.

## 4. Movement

**Scope:** Dash with charges and recharge, client-started movement, server validation and reconciliation, Evade, Movement Validation with Last Valid Position, and strafing and dashing dummies. Each Ability's Dash compatibility is read from its data.

**Acceptance checks:**

- With no movement input, a Dash goes straight back relative to the aim direction.
- A Dash without a charge, or while Rooted or Stunned, is rejected and reconciled.
- Evade begins only when the server accepts the Dash, and projectiles pass through without effect and report `Evaded`.
- A clear teleport returns the player to their Last Valid Position, while one lag spike does not.

**Test in Studio:** Dash in every direction, while firing and on the ledge. Judge distance, speed, Evade timing and readability, whether the strafing dummy is fair to hit, and whether simulated latency causes rubber-banding.

**Left for later tickets:**

- Rooted, Stunned and Frozen are server-set character attributes that only block a Dash so far. Whatever first applies one must also stop walking, and Stunned must also stop Casts.
- `dashCancelsWindup` has no effect yet, because nothing has a wind-up.
- A player's Evade has only been tested by the unit path and dummies. A real player shooting a player belongs to Ticket 8's Stage 2 run.

## 5. Tidal Bolt

**Scope:** the first Ability through the shared projectile path, Ability input and cooldown display, server-owned Wet with Applied By tracking, the Wet visual on the character, and the light push as Knockback under ADR-0002.

**Acceptance checks:**

- Wet appears only after Server Confirmation, and expires at its configured duration.
- The server enforces the cooldown, and a Rejected Cast restores the cooldown display.
- Knockback direction and strength come from the server, and Movement Validation allows for it through `Movement.allowDisplacement`.
- Knockback interrupts an active Dash.
- Evade prevents damage, Wet and Knockback, but does not remove an existing Wet.
- Adding the Ability needed data and only small generic changes, not a separate projectile path.

**Test in Studio:** hit moving dummies. Judge whether Wet is readable at combat distance without any UI, how the push feels, the bolt's speed relative to the Basic Attack, and whether the cooldown suits the fight's rhythm.

**Left for later tickets:**

- Being pushed and made Wet can't be felt solo, because no dummy shoots back. A real player hitting a player belongs to Ticket 8's Stage 2 run.
- A client that ignores a Knockback isn't noticed yet. ADR-0002 calls that suspicious, but Movement Validation only catches moving too far, not too little.
- The Roblox default `Health` script still runs on characters and heals them from the first moment. Ticket 7's Natural Recovery must replace it. Done in Ticket 7, with an empty script of that name in `StarterCharacterScripts`.

## 6. Arc Burst

**Scope:** the short-range Lightning cone, the shared Reaction system driving the Wet Reaction from `ReactionRules`, and payoff feedback for the attacker, victim and spectators.

**Acceptance checks:**

- Arc Burst uses the Hitbox for cone checks, and the server decides the hit.
- Arc Burst on a Wet target triggers the Reaction and consumes Wet, as its data says. Without Wet it deals base damage.
- Applied By and Triggered By are both tracked, and the triggering hit gets the damage and kill credit.
- The Combat code has no Wet-specific or Arc Burst-specific Reaction logic.
- The Basic Attack never triggers the Reaction.

**Test in Studio:** apply Wet, close the distance and fire Arc Burst. Judge whether the payoff feels strong, whether a watcher can tell what happened, whether the VFX hides the next decision, and whether the cone's size and range feel fair.

**Left for later tickets:**

- A cone is decided from the server's positions when the Cast arrives, with no rewind, as ADR-0001 sets out. How that feels under latency, and a real player's Arc Burst on a real player, belong to Ticket 8's Stage 2 run.
- Applied By only travels in the `CombatResult` as `reactionAppliedBy`, and `killed` with `attackerId` names the kill's credit. Nothing reads them yet: Ticket 7's session logs and Kill Recovery should.
- `RemoveStack`, `Replace` and `Leave` are only tested in specs, because no Reaction's data uses them yet.

## 7. Fight loop

**Scope:** health, Natural Recovery, Kill Recovery, death and respawn, the remaining feedback (Kill, low health, killer identification, directional damage indicator) and session logs.

**Acceptance checks:**

- Natural Recovery starts after the configured delay, and damage stops it and resets that delay.
- A player kill restores about 25% of maximum health, credited to the triggering hit.
- Death and respawn return a player with a clean Kit and no Statuses.
- Session logs record fight duration, attack attempts, confirmed hits, Ability usage, Reactions, Dash and Evade counts, and kills and deaths.
- Low-health and screen effects have configurable intensity, and do not pulse excessively.

**Test in Studio:** fight dummies through several deaths. Judge how long a fight takes against the 6–9 second target, whether a disengage resets it, whether you can tell what killed you, and whether the low-health state helps.

**Left for later tickets:**

- Dummies count as players for Kill Recovery through `Tuning.dummy.grantsKillRecovery`, so that it can be felt solo. Monsters must not, and real player-on-player kills belong to Ticket 8's Stage 2 run.
- A player can reset their character from the Esc menu to deny a kill and clear their cooldowns. The session log records it as `cause=NotCombat`, but nothing prevents it yet.
- Session logs print to the server console. Reading them from a real server, and how much survives a server shutting down, belongs to Ticket 8.

## 8. Test readiness

**Scope:** fixes from a Stage 2 multi-client test in Studio, then a published private place ready for Stage 3.

**Acceptance checks:**

- The Stage 2 list in the playtest plan has been run on 2+ clients, and its problems are fixed or written down.
- What each player sees has been compared under simulated latency.
- Testers can join the published place, and session logs can be read from a real server.
- A checkpoint is recorded above.

**Test in Studio:** run the multi-client test with the developer watching each client, then join the published place from a real client. Stage 3 itself, with outside testers, is the only stage that can pass the prototype.

**Stage 2 run, 2026-11-07, two clients through the MCP:**

- Passed: real Q and E between players, Wet, Knockback, Reactions and kills on both clients and the server; private damage numbers; forged and flooded requests; a Dash forced into rejection snapping back; a real Dash's Evade on both clients; a spectator seeing a Reaction; a player leaving mid-fight; effects cleaning up.
- With 200–250 ms of `IncomingReplicationLag`: no corrections or rubber-banding, and client and server agreed on position afterwards.
- Fixed: the kill notice showed Studio's negative test ids as "Killed by -1".
- Fixed: Roblox's random spawns could put players on the same pad. The [spawn rule](first-combat-prototype.md#spawn-rule) replaces them, so `Characters` now loads every character itself, with `CharacterAutoLoads` off.

**Real client, 2026-11-07:** the developer joined the published place from the Roblox app, through its direct link, as private experiences aren't listed anywhere. Session logs read correctly from the real server through F9 → Log → Server, with no errors or warnings at about 58 ms ping. That console can't be copied from, so take a screenshot.

**Closed 2026-11-07, with tester access open:** the developer closed this ticket without the "Testers can join" check. Nobody but the developer has joined the published place, because there are no testers yet. The place is still set to Private, and the [Stage 3 run sheet](stage-3-run-sheet.md) gives the access change to make and test first.

**Written down, not fixed:**

- A player's JSON summary only prints when they leave or the server closes, and hasn't been read from a real server. Whoever's still in the server can read it, but once the last player leaves, its console is gone. In Stage 3, the developer should stay in until the others have left, and read the per-fight lines as they go.
- `IncomingReplicationLag` delayed remote events by the full amount, but other players' movement by only about 30 ms. So this run understates how far behind a laggy player sees targets, and whether cones feel fair under latency, as ADR-0001 asks, is for Stage 3 to show.
- Each client ran at about 15 fps with its window in the background, and judging feel by hand in this setup isn't reliable.
- A malformed request logs a warning each time, so a client could flood the server's console with them.

## Combat changes before Stage 3

On 2026-11-07, after Ticket 8, the developer changed the first loop before any Stage 3 test: a melee Basic Attack, flat Sectors instead of cones, and Charging for Tidal Bolt and Arc Burst. The [first combat prototype](first-combat-prototype.md) and [ADR-0006](adr/0006-server-timed-charging-with-a-two-stage-cast.md) hold the design. Version 26 is the fallback checkpoint from before these changes.

## 9. Flat sectors and melee

**Scope:** the shared flat Sector hit check, Arc Burst moved onto it at its current tap values, the melee Basic Attack, the shooting dummy on tap Tidal Bolts, and renaming `Cone` and `Cones` to `Sector` and `Sectors` wherever the old term is no longer accurate.

**Acceptance checks:**

- One shared Sector check decides both Arc Burst and Basic Attack hits, from the horizontal aim direction, range, width and height allowance, with line of sight. Camera pitch doesn't change it.
- The Basic Attack hits every valid target within 8 studs, 120° and ±5 studs, for about 6 damage, at the server's 0.4 s cadence, with no Knockback, no Status and no Reaction.
- Arc Burst hits within 18 studs, 90° and ±8 studs, and still triggers the Wet Reaction.
- Hit Tolerance is unchanged.
- The shooting dummy fires tap Tidal Bolts through the normal path, at their normal cooldown.
- No code or glossary still calls a Sector a cone.
- Specs cover the Sector's range, width and height edges, pitch being ignored, and cover.

**Test in Studio:** swing at the static and moving dummies, aiming up and down, and use Arc Burst across the raised area's edge. Judge whether 8 studs feels short but fair, whether swings read clearly, and whether a flat Arc Burst ever misses something you expected to hit.

**Built 2026-11-07, not yet checked by the developer or published.** `HitboxRules.sectorHits` is the one Sector check. Server `Cones` became `Sectors`, and client `CosmeticCones` became `CosmeticSectors`. Both Arc Burst and the Basic Attack now have `form = "Sector"`, with `range`, `widthDegrees` and `heightAllowance` stats. 155 specs pass, including a new `SectorsSpec`, which runs a copy of the real module against temporary walls.

**Checked through the MCP in Studio:**

- A swing hit the static dummy for 6, with no Knockback and no Reaction, whether aimed level, steeply up or steeply down. Aimed away, it missed.
- From 9.5 studs away the swing hit, and from 10.2 it missed, against 9.8 for 8 studs plus the Hitbox radius and Hit Tolerance. Aimed 50° off the dummy it hit, and 90° off it missed.
- 34 requests in 2 s gave 6 hits and 28 `OnCooldown` rejections. Holding the real mouse button for 2 s gave 6 hits.
- Arc Burst hit from 19.4 studs and missed from 20.2. Tidal Bolt, then a swing, then Arc Burst gave 12, then 6 with no Reaction, then 30 with the Wet Reaction. So the swing neither triggers nor consumes Wet.
- The shooting dummy fired a Tidal Bolt about every 3 s, each hitting for 12 with Knockback 25.
- A click created the swing's 8 arc segments, which cleaned themselves up. The MCP never caught one on screen, as it only lasts 0.12 s.

**Left for later tickets:**

- How the swing looks and sounds hasn't been seen by a person yet. It's a level arc at 75% of the range, with the old Basic Attack sound renamed `Swing`.
- Arc Burst's bolts are now drawn level, with no tilt, to match the flat Sector.
- From the floor beside the raised area, its ledge blocks line of sight to the shooting dummy in the middle. Arc Burst can't hit it there, and the dummy doesn't fire at a player standing there. This is cover working as intended, but it's worth judging whether it feels right.
- A Sector's height allowance is measured from the Cast's origin, 1 stud above the root, and Hit Tolerance doesn't widen it. Arc Burst's ±8 was set from the dummies' positions, not a measured audit of the arena's heights.
- The Basic Attack keeps `usableDuringDash`, as the old ranged one did.

## 10. Projectile line of sight

**Scope:** stop a projectile hitting a character who is fully behind cover. First reproduce the through-cover hit where practical, then require line of sight from the projectile's position to the target's sight points before a hit counts.

**Acceptance checks:**

- The through-cover case is reproduced before the fix, or the reason it couldn't be is recorded.
- Specs cover a tap-sized Tidal Bolt blocked by cover, a full-size bolt blocked by cover, an exposed target still hit, and a partly exposed target following the existing sight-point rules. The larger case sets the projectile radius directly, since Charging doesn't exist yet.
- Wall collision still uses the projectile's centre path, and Hit Tolerance is unchanged.

**Test in Studio:** stand just behind thin cover while the shooting dummy fires, then lean out past its edge.

**Built 2026-11-08, not yet checked by the developer or published.** A new server module, `Cover`, owns the arena's cover. `Cover:strike` is the centre-path wall raycast, and `Cover:canSee` is the sight-point check that used to be a local `hasLineOfSight` in `Sectors`. `Main` builds one from `workspace.Arena` and passes it to both `Projectiles` and `Sectors`, so cover means the same to both. Every projectile goes through `Projectiles`, so the fix covers players, the shooting dummy and future projectiles. `Projectiles.start` also takes an optional `heartbeat` signal, so specs can step it. 164 specs pass, including a new `ProjectilesSpec` that runs a copy of the real module against temporary walls.

**How a projectile hit is decided now:** a step's path must still come within the projectile's radius, plus the Hitbox radius, plus Hit Tolerance (still 0.3), of the Hitbox. Wall collision still uses only the centre path. On top of that, from some position on that step where the projectile is within that reach, at least one of `HitboxRules.sightPoints` must be clear of cover. It tries the closest position to the Hitbox first, then positions at most 1 stud apart. Two details make this more than "check from the projectile's position":

- Checking only from where a step starts missed real hits when a frame was slow: one step could start hidden behind a corner and end past the target it rounded the corner to reach. With Studio in the background this happened in the real arena. Checking along the step keeps the module's promise that speed can't cause tunnelling.
- When a step ends on a wall, sight is checked from 0.1 studs short of its face, because a ray starting on a part's surface misses that part. Without the gap, both blocked-by-cover specs fail.

`HitboxRules.closestShare` is new: a pure helper that gives how far along a path it comes closest to a Hitbox.

**Reproduced before the fix**, with a copy of the unchanged `Projectiles` in the real arena:

- A tap bolt (radius 0.6) into a temporary 0.5-thick wall hit a target pressed behind it, because its reach of 2.4 extends 0.4 past the wall. Into a 1-thick wall it missed.
- A full-size bolt (radius 1.8, set directly) into a 1-thick wall hit a target pressed behind it, because its reach of 3.6 extends 1.1 past the wall.
- Every cover part in the arena is at least 3 studs thick, so neither bolt reaches through one face-on. Through-cover hits there are a corner problem. A full-size bolt flying past `CoverCentre`'s end came within reach of a target hidden behind it about 2 studs before rounding the corner. A tap bolt's reach is too short for that at this arena's cover.

**Checked in Studio through the MCP after the fix:**

- The same copy, against the same walls, gave no hit through either wall and still hit with no wall.
- `ProjectilesSpec` covers a tap bolt blocked by thin cover, a full-size bolt blocked by cover, both sizes hitting an exposed target, a head above low cover, a shoulder past a wall's end, a target hidden the whole time it's in reach, a slow server frame, and cover beside a full-size bolt's path not stopping it.
- Mutation checks: with no sight check, the three blocked specs fail. Checking only from a step's start fails the slow-frame spec. Checking only the closest position fails the shoulder spec. Removing the 0.1-stud wall gap fails both blocked-by-cover specs.
- Live bolts from the running game's `Projectiles`, fired at the real player through a temporary hook, since removed:

  | Case | Result |
  |---|---|
  | Tap bolt, exposed | Hit for 12 |
  | Tap bolt, 0.5-thick wall pressed in front | No damage |
  | Full-size bolt, 1-thick wall pressed in front | No damage |
  | Full-size bolt, shoulder showing past a wall's end | Hit for 12 |
  | Full-size bolt, hidden behind a wall the whole time it was in reach | No damage |

- On `RampRaised`, the shooting dummy still hit the player every 3 s for 12.

**Left for the developer and later tickets:**

- The arena has no thin cover, and tap bolts couldn't hit through its 3-stud cover before this fix. So the Studio test can confirm that leaning out still gets you hit and that nothing feels off, but it can't show the fix itself. That needs a full Charge from Ticket 11 at a corner, or a temporary thin wall.
- The shooting dummy only fires when it can see the target's middle, with its own raycast. It doesn't use `Cover` or the sight points, so it won't fire at a player showing only a head or a shoulder, though its bolt could hit them. That is cover-respecting, so it was left alone.
- A projectile with its centre path just past a wall's edge can still hit someone partly behind it, if a sight point is clear from where it is. The design intends this: "a genuinely exposed part of them still counts".

## 11. Charging foundation and Tidal Bolt

**Scope:** Charging as a reusable mechanism for any Chargeable Ability, built through Tidal Bolt: the ADR-0006 start and release protocol with attempt identifiers, server-derived Charge, the Tap Window, the exclusive combat state, slowdown, cancels and interrupts, the input rules, the fresh-press rule, auto-release, expiry, session logging of end causes, Tidal Bolt's scaling, its orb Telegraph, the full-Charge cue, the caster's local estimate for immediate visuals, and the server's Charge in replicated Casts. No Arc Burst-specific presentation.

**Acceptance checks:**

- Nothing about Charging is hard-coded to Tidal Bolt: an Ability's data makes it Chargeable.
- The server derives Charge only from its own timestamps, and never accepts a client Charge or duration.
- A release within the Tap Window is exactly 0% Charge, with no slowdown or Telegraph.
- The server rejects a start while on cooldown, and the cooldown starts only on a Cast.
- Dash, Knockback, Stun, Frozen and death end Charging with no cooldown, and ordinary damage doesn't. A `DashRequest` ends it as `Dash`, accepted or not.
- Holding the key after a lost Charge does nothing until a fresh press.
- The client auto-releases at about 1.75 s, and the server expires a Charge at about 2.25 s with no Cast.
- A stale or late release is a Rejected Cast, and can't affect a newer attempt.
- Walking drops to about 12 studs per second only while visibly Charging, and Movement Validation doesn't correct an honest player across the transitions.
- Session logs count each end cause, and Dashes while chasing.
- Tidal Bolt scales linearly to radius 1.8, speed 65, Knockback 40 and damage 15, with Wet unchanged.

**Test in Studio:** tap, partly Charge and fully Charge Tidal Bolt at the dummies, Dash out of a Charge, get interrupted by the shooting dummy, and hold past the maximum. Judge whether the slowdown feels fair, whether the orb reads its size, and whether a full Charge feels worth it without being automatic.

## 12. Charging Arc Burst

**Scope:** Arc Burst on Ticket 11's Charging system, with no duplicated Charging logic: range, width and damage scaling with the Reaction bonus unchanged, the lightning Telegraph, the full-Charge cue, and the caster-only ground preview, which expands with Charge and approximately follows the ground.

**Acceptance checks:**

- Arc Burst scales linearly to 28 studs, 120° and 16.1 damage, and the Reaction bonus stays 16.
- The preview shows the same Sector the server uses at the current Charge, and only the caster sees it.
- The server's Charge decides the Cast, for the caster and for everyone else.
- The burst limits hold at every Rank: about 42 for Rank 1 taps, and at most about 60 for full Charges at Rank 3.

**Test in Studio:** Charge Arc Burst near ledges and the raised area. Judge whether the preview matches what gets hit, whether the lightning reads without hiding Wet, and whether a 28-stud Arc Burst undercuts closing distance.

## 13. Test readiness for the combat changes

**Scope:** Stage 1 and Stage 2 re-run against the changed loop, then a publish by the developer and a recorded checkpoint, with version 26 kept as the fallback.

**Acceptance checks**, verified on 2+ clients where it involves another player:

- the melee Basic Attack, the flat Sector Arc Burst and Tidal Bolt's line of sight
- tap determinism, Charging, and the slowdown starting and ending
- Movement Validation across slowdown transitions
- Charge interrupts, a rejected Dash while Charging, stale releases and expiry
- both Telegraphs, the full-Charge cues and the ground preview
- the server's Charge value agreeing across clients
- kills, deaths and respawns still working
- the forged and flooded request checks still holding, now including `ChargeStart` and release messages

**Test in Studio:** the multi-client run with the developer watching each client, then the developer publishes and the checkpoint is recorded above.
