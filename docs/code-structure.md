# Code structure

The structure serves the first combat loop: **Basic Attack → Dash → Tidal Bolt → Arc Burst → Wet Reaction**. Anything that does not help implement, test or observe that loop probably does not belong in the first milestone.

[Milestone 2](milestone-2-tickets.md) adds the run progression loop, so systems for Kits, Pickups, the Match, Rift Caches and Sentries now belong, each added by its ticket. A general monster system and an inventory still don't. [Ticket 25](milestone-2-tickets.md#25-match-milestones) (Milestones, staged and not built) would add a pure `MilestoneRules`, a server `Milestones` module and a shared combat-idle signal, with Riposte and Catch Breath rules beside them. Its gated Stage 3 Reserve is **one extra field on the Kit**, not an inventory. [Ticket 26](milestone-2-tickets.md#26-rift-collapse-visuals-and-atmosphere) (Collapse presentation, staged and not built) would add a pure `CollapseVfxRules` and one client `CollapseVfx` controller, with no new remotes and no server code. [Ticket 27](milestone-2-tickets.md#27-minimap-and-objective-awareness) (Minimap and Tactical map, staged and not built) would add a pure `MinimapRules`, one client controller and one baked layout data module, with no new remotes, no server code and no new replication.

## Layout

```text
ReplicatedStorage
└── Shared
    ├── Environment
    ├── Config
    │   ├── Abilities
    │   ├── Statuses
    │   ├── Reactions
    │   └── Tuning
    ├── Rules
    │   ├── ReactionRules
    │   ├── RankRules
    │   ├── KitRules
    │   ├── PickupRules
    │   ├── MatchRules
    │   ├── MapRules
    │   ├── VoidRules
    │   ├── LootRules
    │   ├── CacheRules
    │   ├── SentryRules
    │   ├── WardenRules
    │   ├── MonsterRules
    │   ├── MilestoneRules
    │   ├── RiposteRules
    │   ├── PveMotionRules
    │   ├── PveThreatRules
    │   ├── PveActionRules
    │   ├── ShieldRules
    │   ├── HookRules
    │   ├── HitboxRules
    │   ├── CastRules
    │   ├── ChargeRules
    │   ├── DashRules
    │   ├── MovementRules
    │   ├── StatusRules
    │   ├── KnockbackRules
    │   ├── HealthRules
    │   ├── SessionRules
    │   └── SpawnRules
    └── Net
        └── Protocol

ServerScriptService
└── Server
    ├── Main
    ├── Characters
    ├── Kits
    ├── Pickups
    ├── Match
    ├── MapLayout
    ├── Caches
    ├── Sentries
    ├── Wardens
    ├── WardenChecks
    ├── Collapse
    ├── CollapseChecks
    ├── Shields
    ├── Dummies
    ├── Casts
    ├── Charging
    ├── Cover
    ├── Projectiles
    ├── Sectors
    ├── Combat
    ├── Statuses
    ├── Movement
    ├── Recovery
    └── Session

StarterPlayer
├── StarterCharacterScripts
│   └── Health (deliberately empty, turning off Roblox's default healing)
└── StarterPlayerScripts
    └── Client
        ├── Main
        ├── Input
        ├── Camera
        ├── Dash
        ├── Knockback
        ├── ForcedMovement
        ├── Casting
        ├── CastAnimations
        ├── ChargeTelegraphs
        ├── CollapseTelegraph
        ├── CosmeticProjectiles
        ├── CosmeticSectors
        ├── PickupVisuals
        ├── PveVisuals
        ├── PveTelegraph
        ├── MonsterVisuals
        ├── Replacing
        ├── Spectating
        ├── StatusVisuals
        ├── ShieldVisuals
        ├── ReactionVisuals
        ├── Feedback
        ├── HealthDisplay
        ├── Hud
        └── MatchHud

ServerStorage
├── Tests
│   ├── RunAll
│   └── <Name>Spec
├── Tools
│   └── RiftCrownGreybox (an Edit-mode tool that rebuilds the map, never required by the game)
└── MatchMapGreybox (the Ticket 18 map, kept as a reference only)
```

The server creates a `Remotes` folder under `Shared/Net` at startup, from the events defined in `Protocol`, and a `Pickups` folder in the workspace for the Pickups' Parts. A Match's first setup creates `Caches` and `Sentries` folders for their models, so they never exist with the Reference Kit. Each client draws Pickups into its own `PickupVisuals` folder, and Cache and Sentry effects into `PveVisuals`. Avoid dots in instance names, because the MCP reads them as path separators.

The workspace holds three static environment folders:

- `Workspace.Arena`: the Reference Kit's arena, used for Stage 3, with pads `Spawn1`–`Spawn4`.
- `Workspace.MatchMap`: the Rift Crown, from [Ticket 21](milestone-2-tickets.md#21-rift-crown-arena-layout), about 260 across and centred at (600, 0, −450). Under it:
  - `Sections`: a Model for each of the 15 Sections (`Shard<n>Outer`, `Shard<n>Inner`, `CrownOuter`, `CrownInner`, `CrownDais`), with `SectionId`, `CollapseLayer`, `CollapseParent`, `ShardId`, `Adjacent`, `ShardAzimuth` and `GateAzimuth`.
  - `Connectors`: a Model for each spoke, pair link and rim link, and a marker Part for each Dash shortcut, with `ConnectorType`, `ConnectorRole`, `CollapseLayer`, `FromSection`, `ToSection`, `Width` and `Gap`.
  - `Markers`: the invisible, non-colliding setup Parts. `CacheHome<n>A`/`B`, `CacheForward<n>` and `CacheCrown1`–`3` carry `CacheKind`, `Sentry1`–`3` are found by name, `WardenSlot1`–`3` and `EliteSlot` carry `SlotKind`, and there is one `CollapseCenter`. Each names its `SectionId`.
  - `Pads`: the six invisible `SpawnLocation`s `MapSpawn1`–`6`, in order around the ring, each with its `ShardId`.
- `Workspace.Hub`: the temporary Hub's greybox, with pads `HubSpawn1`–`HubSpawn4`.

`Shared/Environment` is the one list of those folders, and every environment query uses it: cover, sight, camera collision, cosmetic impacts and floors. Runtime models such as Caches, Sentries and Pickups are never environment.

## Rules for each part

- **`Config`** holds data that both client and server are allowed to know.
- **`Rules`** holds deterministic, pure logic where practical, free of Roblox object dependencies.
- **`Main`** on each side only requires and starts systems, provides explicit dependencies and establishes startup order. It does not accumulate gameplay logic.
- **Shared code** does not hold security-sensitive decisions. The server remains authoritative over outcomes.

## Kits

From [Ticket 15](milestone-2-tickets.md#15-per-player-kit), each player's Kit is owned by the server.

- **`KitRules`** is the pure logic: building the Starting Kit, finding an Ability's slot, the first free slot, equipping into an empty slot with the Equip Lock and the Rank cap, whether a Cast is allowed, and encoding the Kit as attributes and back. From Ticket 16 it also decides fits (`fit`: Equip, Upgrade, AtCap or NeedsReplace), `upgrade` and `replace`. It is the only place that enforces the gameplay Rank cap, `Tuning.kit.maxRank`, while `RankRules.MAX_RANK` is only what the Ability data supports.
- **`Kits`** (server) holds each player's Kit, gives each new character the Starting Kit from `Tuning.kit`, and is the only writer of the `KitSlot<n>Ability`, `KitSlot<n>Rank` and `KitSlot<n>LockedUntil` attributes on the `Player`. `Kits.canCast` is the only gameplay source of an Ability's slot, Rank and whether it may be Cast, and `Casts` asks it for every request. `Kits.grant` puts an Ability into the first free slot, for tests. `Kits.collect` and `Kits.replace` are how Pickups change a Kit, and each decides the fit again from the Kit as it is then.
- **Clients** rebuild a copy of their Kit from the attributes with `KitRules.fromAttributes`, for the HUD, local prediction and drawing only. `Input` reports slots, and `Casting` maps them to the Abilities in the copy. Other players' Telegraphs read the `ChargingRank` character attribute that `Charging` sets. No Rank from a client is ever used.
- **Cooldowns** stay in `Casts`, per Ability per life, separate from the Kit, so swapping an Ability out and back in never resets one.

## Pickups

From [Ticket 16](milestone-2-tickets.md#16-pickups-and-replacement), Ability Pickups are owned by the server.

- **`PickupRules`** is the pure logic: who collects a Pickup (`chooseCollector`), the checks on a replacement before the Kit's own (`validateReplace`), which Pickup a full Kit is offered (`chooseReplaceTarget`), the token bucket that guards `ReplaceRequest`, where a dropped Ability lands, and the supported Pickup shapes.
- **`Pickups`** (server) keeps each Pickup's record and gives it an invisible, tagged Part in `workspace.Pickups` carrying `PickupId`, `PickupAbility`, `PickupRank` and `PickupSpawnedAt`. About 10 times a second it gives each Pickup, in ascending id order, to the nearest living, unblocked player it fits, through `Kits.collect`. It handles `ReplaceRequest` in one synchronous step, so the first valid request wins, and drops the replaced Ability at the player's feet as a new Pickup. Either way it fires `PickupCollected` and removes the Part. There are no loose Pickups at a Match's start: Abilities enter the world from broken Rift Caches, replacements and Eliminated players' Kits, all through `spawn` and `drop`.
- **Clients** draw Pickups in `PickupVisuals`, from the tag and attributes only. `Replacing` keeps its own copy of the Kit, offers the nearest Pickup a full Kit could take only by replacing, shows the prompt, and sends `ReplaceRequest` when the player holds the modifier and presses a slot. `Input` sends a slot's press to `Replacing` instead of `Casting` while the modifier is held. Nothing on the client predicts a Kit change.

## Match

From [Ticket 17](milestone-2-tickets.md#17-one-life-match), an Empty Starting Kit runs the one-life Match. With the Reference Kit none of this is installed, and the arena behaves as before.

- **`MatchRules`** is the pure logic. It holds the Match's state as plain data: the phase (Waiting, Countdown, Running, Results), the roster with each player's Alive or Eliminated status and history, and the pending end. It also decides:
  - the phase steps (`step`)
  - the end (`checkEnd`, the only place that does, including the short end grace)
  - eliminations, Return to Hub and spawn dealing
  - the gates (`canAct`, `canCollect`, `canBeHit`)
  - the result rows
- **`Match`** (server) is the only owner of participation.
  - It runs `MatchRules.step` on a short loop, and immediately on each elimination.
  - It does what each step means in the world: clearing Pickups, setting up and clearing Caches and Sentries, resetting Kits, loading characters onto the map or into the Hub, the whole-Kit drop, removing bodies, and cancelling Charges at the end.
  - It handles `ReturnToHub`.
  - It installs its policy and gates into other systems at start:
    - the spawn policy, death listener and target filter into `Characters`
    - `canCollect` into `Pickups`
    - `canAct` into `Casts`
    - the leave handler into `Kits`
  - It publishes the presentation attributes `MatchPhase`, `MatchPhaseEndsAt`, `MatchId`, `MatchAliveCount` and each player's `MatchState`. The server never reads these back.
- **`Characters`** keeps the spawning mechanics: pads by area, `spawnInArea`, `removeCharacter` and `lastPosition`. Administrative replacements never count as deaths. It reports real deaths and losses through `onDeath`.
- **`Kits`** gives the Match `takeAll` (a transfer, for the drop), `resetForMatch`, and `onPlayerLeaving`, which runs before a leaving player's Kit is forgotten.
- **`Session`** snapshots counters at Match start, so a Match's own numbers are the difference (`SessionRules.matchStats`). It also logs `MatchStart`, `Eliminated`, `MatchEnd`, `MatchResult` and `ReturnToHub`.
- **Clients** only present the Match, from its attributes and remotes. `MatchHud` shows the status line, the elimination feed, the player's own elimination banner, the end banner and the results, and dims the slots when the player can't act. `Input` passes no slot, Basic Attack or replacement presses on then. `Spectating` moves the camera onto living players, through `Camera.setSuspended`, and owns Q and E through `Input.setSlotOverride` while it's active. From the Hub, **G** (`Input.onSpectatePressed`) or the button starts and ends watching a Running Match, because the Hub's mouse is locked to the centre. The only thing it sends is `ReturnToHub`.

## The Rift Crown

From [Ticket 21](milestone-2-tickets.md#21-rift-crown-arena-layout), the Match's map carries its own structure as data, and the server reads it rather than knowing the layout.

- **`MapRules`** is the pure logic: the layout as plain data, `validate` (eight numbered structural rules, returning the problems), `hasInwardRoute` and `shardOfSection`.
- **`MapLayout`** (server) reads a map folder into that layout, answering what needs the world by raycasts against that folder alone: the floor under a marker, its lip margin, aprons and the lowest underside. It only ever reads, and takes any root, so a map can be checked while it is built beside the live one. `get`, `problems`, `sectionAt` and `shardAt` are for the live map.
- **`VoidRules`** is the displacement credit for a fall and where a Kit lost to the Void lands.
- **`Match`** owns what happens on the map:
  - the roster cap and the spread of spawns, from `MatchRules`
  - the **arrival hold**: every character is anchored on its pad until GO, and `MatchRules.arrive` makes GO the Match clock's origin, which `Session`, `Caches` and `Sentries` are told
  - the **Void**: anyone below `Tuning.map.voidKillY` after GO is Eliminated with the cause `Void`, credited to the last player whose hit pushed or pulled them, which `Combat` reports through `setDisplacementListener`
  - the **Crown seal**'s opening, through `Caches.update`
- **`Caches`** holds each sealed Cache's own opening time and refuses every hit before it.
- **Clients** read `MatchGoAt` and `MatchCrownOpensAt` on Workspace and `CacheSealed` and `CacheOpenedAt` on a Crown Cache, for presentation only.
- **`ServerStorage.Tools.RiftCrownGreybox`** rebuilds the greybox as `Workspace.RiftCrownBuild` from one table of dimensions. It is swapped in by hand: the old `MatchMap` out, the build folder renamed to `MatchMap`.

## Rift Caches and Sentries

From [Ticket 18](milestone-2-tickets.md#18-match-map-caches-and-sentries), the Match's PvE is breakable Rift Caches and stationary Sentries, owned by the server. Only `Match` sets them up and clears them, so with the Reference Kit none exist and no loop runs.

- **`LootRules`** is the equal Ability bag. It covers:
  - `validate`, which checks the pool divides the drops evenly
  - `buildBag`
  - `deal`: each Shard's own Caches take different Abilities, and the Crown's take the rest, with no Cache repeating one
  - `check`, the rules a deal must hold for any seed

  Equal counts and variety are both strict since Ticket 21: a pool that can't hold them is refused.
- **`CacheRules`** holds the crack stage for a health, where drops go, and `mayDamage`: only a player who is Alive in a Running Match, checked at impact.
- **`SentryRules`** is one Sentry's Idle → Telegraph → Recover cycle as plain data. The aim locks when the Telegraph starts, and the shot is cancelled if the target becomes invalid or out of sight. Sight is passed in.
- **`Caches`** (server):
  - `prepare(random)` deals a bag without changing the world.
  - `setup(deal, matchId)` builds the 21 Caches from the markers. Each is a model with a hidden Humanoid, a root for the fixed Hitbox and a body sized to it, registered as a `cache` target.
  - It owns each Cache's hidden contents, its health, `CacheStage` and `CacheBroken`, the break into a stump that blocks nothing, and the drops through `Pickups.drop`.
  - It logs `LootDeal`, `LootContents` and `CacheBroken`.
  - `Match` gives it the attacker gate.
- **`Sentries`** (server):
  - `setup(matchId)` builds the three Sentries, registered as `sentry` targets with no Kill Recovery and a Knockback handler that does nothing.
  - One loop at about 5 Hz, started by the first setup, steps them, and a delay fires each shot on time.
  - A shot is a Rank 1 tap Tidal Bolt through `Casts.castFromServer`, from the firing point (its trusted, server-only `options.origin`).
  - It publishes `SentryTelegraphEndsAt` and `SentryDead`, and logs `SentryKilled`.
- **Target kinds.** `Characters.registerTarget` takes a `kind` (`dummy`, `sentry` or `cache`), and players are `player`.
  - **`Combat.applyHit`** sends a Cache hit to its own `onHit`, which applies only the damage it allows. Nothing else about the hit happens to a Cache, and the attack stops there either way. A refused hit is sent as a `Blocked` `CombatResult`, which ends the Cosmetic Projectile at the Cache.
  - A Sentry's attack on another Sentry is also `Blocked`.
  - Sentries otherwise take the normal path, with Statuses and Reactions. Their `onKilled` is how `Sentries` learns of a kill.
  - `Recovery` heals neither kind.
- **`Session`** keeps hits by or on a Cache or Sentry out of players' fights, damage, kills and Reactions, through `SessionRules.countsForPlayers`. It still remembers a Sentry's kill, so the `Eliminated` log can say `by=sentry:N`. `Match` passes `PlayerEliminated` and the rows a player killer only.
- **`Match`:**
  - It checks the loot bag once at startup, and disables Matches for good with one error if the bag is uneven.
  - It deals the bag just before a start is due, so a failure changes nothing in the world.
  - **Start:** `clearAll`, then `Caches.setup`, then `Sentries.setup`, then spawns on the `"Map"` pads.
  - **The end of Results** also runs `Caches.clear` and `Sentries.clear`.
- **Clients.** `PveVisuals` draws crack stages, the break burst and sound, the stump, the Sentry Telegraph orb and the wreck, from tags and attributes only. What a Cache holds never reaches a client.

## Rift Hook and Stone Guard

From [Ticket 19](milestone-2-tickets.md#19-rift-hook-and-stone-guard), the two new Abilities extend existing paths rather than adding systems beside them.

- **Forms and Casting Styles.** Both are `Instant`, through the one `CastRequest`. Stone Guard is the new `Self` form: `Casts` delivers it to `Shields.deliver`, skips the `InvalidAim` check for that form only, and the client draws no delivery geometry. `CastRequest` refuses a caster who is `Stunned` or `Frozen`, whatever the Ability, the Basic Attack included.
- **Presentation data.** `visual.castAnimation` (`Release`, `Swing` or none) chooses a Cast's animation, and `visual.projectileStyle` (`Orb` or `Tether`) a Cosmetic Projectile's look. Pickup shapes are Sphere, Diamond, Cube and Cylinder.
- **`ShieldRules`** is the absorption maths (`absorb`, `new`, `isActive`, `broke`). **`Shields`** (server) holds each character's one Shield, replaces rather than stacks, expires it quietly, clears it on death through `Characters`, and is the only writer of `Shield`, `ShieldMax` and `ShieldEndsAt`.
- **`Combat`** runs each hit's combined damage, the Reaction's bonus included, through `Shields.absorb` once, after Evade and the Reaction and before Health. `CombatResult.damage` is Health lost, with `absorbed` and `shieldLeft` when the target had a Shield. Session keeps `damageDealt` as Health lost and adds a log-only `shieldAbsorbed`.
- **Forced movement.** An Ability with `forcedMovement = "Pull"` gets its Knockback from `HookRules` instead of `pushStrength`: a destination 4 studs in front of the caster's current position, shortened by `Cover:sweep` (a Spherecast against the environment roots, ignoring characters), as a velocity over `Tuning.hook.pullSeconds`. `CombatResult.knockbackSeconds` carries that duration, and the client's Knockback, Movement Validation and dummies use it when present and `Tuning.knockback.durationSeconds` otherwise. A target registered `immovable` (Sentries) is never pulled, and with no living caster there's no pull.
- **Clients.** `ShieldVisuals` draws the shell, its thinning, the break and the quiet expiry from the attributes and `CombatResult`. The server clears the attributes before the hit's `CombatResult` arrives, so it waits 0.15 s before choosing a break or a fade. A movement with its own duration stops on arrival, on the client and for dummies, while a normal push is only timed. `Hud` has a Shield bar above the Health bar. `Feedback` has the Shielded hit marker and the pull Beam.

## Run-test telemetry

From [Ticket 20](milestone-2-tickets.md#20-run-test-readiness), `Session` logs more during a Match, for pacing and the run test's watch list. Outside a Match, as with the Reference Kit, every line is as before. None of it reaches the world, the clients or any attribute.

- **One clock.** `Match` passes `state.startedAt` to `Session.beginMatch`, and every Match time is seconds since Running began on server time, as `CacheBroken` already was. `recordMatchEnd` ends the Match's context.
- **Pacing.** `Pickups` passes the Kit's occupancy (`KitRules.filledCount`) to `recordPickup`, which appends `seconds` and `filled` and sets the player's `firstAbility` and `fullKit` once (`SessionRules.noteProgress`). `matchStatsOf` carries them, so the Match stores them at elimination and a leaver keeps them.
- **Central fights.** `Combat` passes the target's position in the hit report. The hit that starts a fight tags it with the Match, its `at` and `central` (`SessionRules.isCentral` against `Tuning.session.centralRadius`, from the centre of the `Map` pads). `MatchEnd` counts each player's `Fight` line, so a duel counts 2.
- **Watch list.** `Shields.deliver` returns whether it granted a Shield, and `Casts` then calls `recordGuard`. `Combat` calls `recordPull` for a Hook pull that moves a player, with the server's own displacement. `Session` keeps each player's last pull for `Eliminated`'s `hookPulledAgo` and `hookBy`, and clears it at elimination, a fresh character, Return to Hub and either end of a Match. It is never kill credit.
- **`MatchResult`'s extra fields** (`firstAbility`, `fullKit`, `guards`, `shieldAbsorbed`) come from `statsById`, which `Match` passes to `recordMatchEnd` beside the rows. The rows that reach clients don't carry them.
- **Specs.** `Session.start({ manual = true, clock, log, characters })` runs it with fakes, with `handlePlayerAdded`, `handlePlayerRemoving` and `handleCharacterAdded` in place of the Players events (`SessionSpec`).

## Rift Collapse

From [Ticket 22](milestone-2-tickets.md#22-rift-collapse), **built in steps**. Steps 1 to 8 exist, and `Match` now drives the Collapse, but only with an Empty Starting Kit: with the Reference Kit nothing is built and the Match behaves as before.

- **`CollapseRules`** (pure, `Shared/Rules`) holds the schedule and its validation, the ordered event cursor (`peek`, `complete`) with the stall shift and settle window, the replicated state (`publicState`), the phase labels, the HUD presenter (`present`), the QuietSpan rules, Section occupancy and the `CollapseParent` walk. Its config is `Tuning.collapse`.
- **`Collapse`** (server) is a factory, `Collapse.new(deps)`, with the map root, a holding folder, a clock, a telemetry sink, the object hooks and the Workspace-attribute holder injected, so specs run it on a fixture. It owns the Section registry, the `Normal → Warning → Removed` states, atomic layer removal by reparenting into the holding folder, restore and reset, and the integrity check. `Match` drives it, as it drives `Caches` and `Sentries`.
- **`Match`'s part (step 3).** It is handed `Collapse.new` as `collapseFactory` and builds the one instance at startup, only with an Empty Kit. Then:
  - `prepare` (restore and the intact assertion) runs just before the loot deal, and a failure disables Matches
  - `start` runs at GO, `stop` at the end and `reset` after Results
  - `Match.sync` applies what is due. It's called by `canAct`, `canCollect`, the target filter, the Cache attacker gate, `tick`, the Sentry loop and a Heartbeat step, so gameplay before a removal sees the old world and after it the whole layer gone
  - `settleUntil` goes into `MatchRules.checkEnd`, so no verdict lands inside the settle window (`Tuning.collapse.settleSeconds`, 1.75). The end does **not** re-read the window: `pendingEndAt` is fixed when the end becomes pending, as `resolveAt(death + endResolutionSeconds, settleUntil then)`, and a removal applied later, inside the 0.25 s grace, does not extend it. A `pendingHeld` flag records that the settle window (not the grace) set the deadline, for the `deferred` telemetry. `Match.schedulePendingEnd` uses the fixed deadline
- **Attributes and timing (steps 4 and 5).** The Workspace attributes appear at GO and not before. `Collapse:timing(now, crownOpenSeconds)` gives the Eliminated log its `phase`, `sinceRemoval` and `settle`.
- **The Kit-drop fallback.** `Match` leaves a whole-Kit drop alone where its floor stands. Over a consumed Section it asks `Collapse:relocate`, which walks the `CollapseParent` chain and returns live floor on the nearest surviving ancestor, or nothing, in which case the Kit is consumed. The registry's bounds are world-aligned boxes, because a model's own bounding box follows its pivot, which is rotated on the real map.
- **Displacement records.** A Cast's charge band travels `Casts` to `Projectiles` or `Sectors` to `Combat`, which tells the displacement listener the band, the victim's position and the distance given. `Match` stores them in the credit as `hitRadius`, `hitDist` and `chargeBand`.
- **Object hooks.** `Caches.removeSection`, `Sentries.removeSection` (and its `setSync`) and `Pickups.sweepUnsupported` are idempotent, non-yielding and return flat counts. A Cache or Sentry marker names its Section with `SectionId`, and a loose Pickup is judged by its support ray alone.
- **Nothing else holds Section parts.** A consumed model is reparented out of the map, so system code must not cache Section descendants. Steps 3 and 5 grep for it.
- **Client presentation (step 6).** Client-only, no server code, no new remote and no sound.
  - `CollapseRules` also holds the player-facing text table (`TEXT`, with `{named}` placeholders), `statusLine`, `alertLine`, `announceLine`, `pulse` and `arrowAngle`, all pure and specced.
  - `MatchHud` builds its status line from `CollapseRules.present` whenever the Workspace carries `CollapseLayer`, for a player, a spectator and a Hub viewer, and keeps its old lines otherwise. It adds the local danger alert beneath the line (only while the presenter says `alert`) with the arrow, and the Warning announcement. It holds its last presentation for 80 ms after a Collapse attribute changes, because the four attributes arrive a few frames apart.
  - `CollapseTelegraph` gives each Section or connector in `Warning` (its own `CollapseState`, while `CollapseWarning` is true) a Riftlight `Highlight` and a beacon in `Workspace.CollapseTelegraph`, pulsing on its removal time. It also finds the Section the local character stands on and the arrow's direction. Ticket 26 replaces it in place.

- **Telemetry (step 6).** Log lines only, no gameplay or remote.
  - `Collapse` takes an optional `observe` hook from `Match` (the Alive players, for telemetry alone, run in a protected call so it can never fail an event). It adds the Alive fields, `CollapseTick` (`CollapseRules.ticksDue`) and `CollapseExposed`, and keeps the checkpoints, summed loot and health counters for `Collapse:summary`.
  - `Session.setDamageListener` tells the `Match` of damage to players. The `Match` owns the QuietSpan state and the PvP, elimination, Void and Kit-drop counters, and writes `QuietSpan` and `CollapseSummary`.

- **Structural checks (step 8).** `CollapseChecks` (server) is read-only: a pure `evaluate` over plain data for the schedule, parent chains, holding folder, registry, stale attributes and homes, plus a world `read` that finds what would still collide or answer queries around the Dais once Layers 1 to 4 are gone, and whether the Dais is flat, anchored, self-contained and lethal on every side, all without moving anything. `RunAll` starts with it, `Collapse:assertIntact` runs it when `Match` passes `structure = {}`, and it is run in Edit mode before a `.rbxl` snapshot.

## Rift Warden

From [Ticket 23](milestone-2-tickets.md#23-rift-warden), **built in steps** (steps 1 to 4 exist). With the Reference Kit no Warden exists and no loop runs.

- **`WardenRules`** (pure, `Shared/Rules`) holds the population table and Gate selection (`count`, `select`), the distinct reward deal, the four-phase FSM with its timestamp-driven action (`step`, `noteDamage`), targeting and Threat, attack choice and the locked Swing and Slam, regeneration, the stage gate, the locomotion maths, no-progress, fault and step checks, and config validation. Its config is `Tuning.wardens`.
- **`Wardens`** (server) is a module like `Sentries`, with its dependencies injected so specs run it in manual mode. It owns the records, the `warden:<GateId>` targets, the one Heartbeat, `finish`, `removeSection` and `clear`. `Match` is the lifecycle owner: it calls `setup` with the occupied pads' Gates after the spawn spread, `setGo`, `clear`, and `removeSection` from the Collapse hook.
- **Combat hooks** (Ticket 23 step 2): a target `gate(hit)` refused hit is a `Warded` result, `resistsForcedMovement` gives a `Hit` with `resisted`, `Combat` reads a hit's Ability through `abilityOf` so a Warden's own attacks need no Ability entry, `Sectors.fire` takes `playersOnly`, and `Casts.attackFromServer` delivers a non-Ability attack down the Sector path.
- **Shared primitives (Ticket 24 step 0).** `PveMotionRules`, `PveThreatRules` and `PveActionRules` hold the ground motion maths, recent-damage Threat and the timestamp action record that `WardenRules` and the regular monsters share, and `PveTelegraph` (client) the wedge and disc Telegraphs. `Protocol.isPveKind` is the one PvE kind list. `WardenRules` keeps its names and the FSM. The two motion and Threat modules also list the settings keys they read (`settingKeys`, `missingSettings`), which every user's config must carry under those names.
- **`WardenChecks`** (server, read-only) is the static placement validator, like `CollapseChecks`. The slot markers carry `GateId` and `TerritoryRadius`, and each pad a `GateId`.
- **Not yet built:** `WardenVisuals` (client, from attributes only), the Warded and Resisted cues, the full `WardenSummary` telemetry, and the `source` tag on Pickup log lines.

## Regular monsters

From [Ticket 24](milestone-2-tickets.md#24-regular-monsters), **built** (the rules, the runtime and its `Match` integration, the visuals and the telemetry). With the Reference Kit no monster exists and no loop runs.

- **`MonsterRules`** (pure, `Shared/Rules`) holds the deal (`deal`, with its fixed draw order), the Idle, Engaged, Returning and Dead machine (`step`, `noteDamage`, `kill`), targeting, the goal (a pair offset inside the Territory only, the unshifted edge point beyond it, and `step` measuring the edge hold against the runtime's own `view.goal` when given), no-progress, Fault and config validation. It reuses `PveMotionRules`, `PveThreatRules` and `PveActionRules`. Its config is `Tuning.monsters`.
- **`MonsterChecks`** (server, read-only, step 1b) is the static Home placement validator, like `WardenChecks`. The six `MonsterHome<n>` markers (one per Shard, on its Inner Section) carry `SlotKind = "Monster"`, `HomeId`, `ShardId`, `SectionId`, `TerritoryRadius` and two slot offsets. A monster's reward appears at Home, as a Warden's does. `RiftCrownGreybox` writes the six markers from its `MONSTER_HOMES` table, so a rebuild keeps them.
- **`Monsters`** (server, step 2) is a module like `Wardens`, with its dependencies injected so specs run it in manual mode. It owns the records, the `monster:<Shard>-<slot>` targets, the one Heartbeat, `finish`, `removeSection` and `clear`, and spawns a flagged kill's reward at Home. `Main` starts it and `Match` is its lifecycle owner, as for Wardens: `setup(matchId)` after `wardens.setup` and before anyone is placed, `setGo`, `clear` at Results, `removeSection` from the Collapse hook and `setSync`. With the Reference Kit `Match` never starts, so none exists. Telemetry (step 6) is per-Home counters kept in protected calls, and `Match.endMatch` calls `summary(players)` once, before `clear`, to log `MonsterSummary`.
- **`MonsterVisuals`** (client, step 5) draws each Riftling from its tag and attributes alone: a Part rig, the claw's wedge from `PveTelegraph`, damage stages, the wake, return and death body language and the Cache-break cue on a reward drop. Wet and the other Statuses come from `StatusVisuals.mirror`, shared with `WardenVisuals`. `Feedback` names the claw and a `monster` id.
- **PvE classification (step 6).** `Session` keeps a Riftling's death out of players' kills, damage and hits, as for Sentries and Wardens, and still names a Riftling that kills a player. In `Combat`, an attack by a Sentry, Warden or Riftling on another of them is `Blocked`. A Pickup from a Warden or a Riftling carries a generic `sourceId`.

## Rift Collapse presentation

Specified in [Ticket 26](milestone-2-tickets.md#26-rift-collapse-visuals-and-atmosphere), **not built**, and staged. Module names and folders are provisional and recorded when built. It is client-only presentation over state [Ticket 22](milestone-2-tickets.md#22-rift-collapse) already replicates, so it changes no authority.

- **Pure rules.** A `CollapseVfxRules` holds stage derivation, tiers, pulse phase and rate, budget distribution and the essentials floor, the drop order, context derivation and configuration validation. It never touches `Workspace`, and `RunAll` covers it.
- **One client controller.** `CollapseVfx` is state-derived and edge-detected. It reads Collapse state through one `matchId`-keyed accessor, and one scope per Match and role owns every effect, tween, sound, connection, callback and Lighting claim. Teardown is destroying the scope.
- **Replaces Ticket 22's minimal telegraph in place,** in the same module and on the same attributes, with no second controller and no duplicate warning effects or audio.
- **No new RemoteEvent and no server code.** The only authored data is the Ticket 21 presentation metadata (`CutLine`, `VfxGroup`, `BeaconAnchor`), generated once at authoring time. Effect parts are `CanCollide`, `CanQuery` and `CanTouch` false and live outside the authored Section folders.
- **Lighting is client-owned,** through Ticket 26's own effect instances, with compare-and-restore for any shared property. The replicated Collapse state is global, matching the one-Match constraint, and the accessor is the only preparation for concurrent Matches.
- **Diagnostics** are concise `[Riftfall][CollapseVfx]` console lines, with no per-frame logging.

## Minimap and Tactical map

Specified in [Ticket 27](milestone-2-tickets.md#27-minimap-and-objective-awareness), **not built**, and staged: Stage 1 after Ticket 22 is accepted, Stage 2 (the Warden marker) after Ticket 23. Module names and folders are provisional and recorded when built. It is client-only, read-only presentation, so it changes no authority.

- **Pure rules.** A `MinimapRules` holds the projection, the horizontal distance filter, overlap suppression, marker priority, the Collapse state derivation and configuration validation. It never touches `Workspace`, and `RunAll` covers it. It is not `MapRules`, which is Ticket 21's structural validator.
- **One client controller** draws both views from one presentation model. It reads the attributes Ticket 22 already replicates (and Ticket 26's accessor if it exists), `PickupFromElimination` on Pickups and `Spectating`'s target. A consumed Section's model leaves the client's Workspace, so Removed means an authored id whose model is absent while the Collapse runs. Nothing reads or writes a server-only structure.
- **One baked layout data module.** The polygons for the Sections and connectors are generated once from the authored footprints by an Edit-mode tool, keyed by `SectionId` and the connector's identity, and checked against the real map. No live Workspace scan and no `ViewportFrame`.
- **No new RemoteEvent, no server code, no new replication and no map service.** Input is observed through `Input`, which still performs the gameplay action first. One scope per Match and role owns every connection and loop, and destroying it is the teardown.

## Platform input

Specified in [Ticket 29](milestone-2-tickets.md#29-platform-input-and-mobiletablet-support) and [ADR-0010](adr/0010-touch-input-through-one-action-dispatcher-and-a-custom-touch-movement-driver.md), **not built**, and staged. Module names and folders are provisional and recorded when built. It is client-side input and presentation, so it changes no authority and adds no `RemoteEvent`.

- **`Input` stays the one action API.** Keyboard and mouse and touch are sources feeding it, so Casting, Dash, Replacing and Spectating never learn the device. A minimal pure `ActionRules` in `Shared/Rules` holds only the holder, edge and cancel state (one holder per source per action, counted across sources). Key mapping, the replace modifier, the Match gate and the Spectating override stay in `Input`. Characterization specs of today's behaviour come first.
- **Touch is ours, PC is native.** On touch devices a small movement driver owns the floating stick and the Jump button (`DevTouchMovementMode = Scriptable`, `Humanoid:Move` from the camera's yaw), and a touch router owns every touch by its `InputObject`. PC and gamepad movement stay Roblox-native. `Camera` takes one summed look delta from `Input`.
- **Pure rules** (provisional): `TouchRules` (hit testing, tap and drag, the hold-confirm state machine, stick math), `TouchLayoutRules` (rectangles from a profile, the viewport and the insets, with overlap specs) and `ContextActionRules` (priority, the stable-time latch, the tuple-bound hold). `Tuning.touch` holds the numbers.
- **One input profile module** owns Desktop, Phone or Tablet, and the HUD modules subscribe to it. Its detector watches touch independently of the touch router.
- **Existing client modules touched:** `Input`, `Camera`, `Casting` (one per-slot cancel handler), `Replacing` (the card path with a locked Pickup id and a Match-phase check), `Hud`, `MatchHud` and `Spectating`. No server script and no `Protocol` event.

## Dependencies

Systems receive their dependencies explicitly, rather than reaching around the DataModel for each other, and circular ModuleScript dependencies are avoided. No dependency-injection framework is needed. For example:

```text
Combat.start({
    statuses = Statuses,
    movement = Movement,
})
```

## Networking

`Net/Protocol` defines event names and expected payload shapes in one place, so all client and server traffic can be reviewed there. Every value arriving from a client is untrusted and its payload is validated.

The exact events can change, but the first loop will likely cover:

- `CastRequest`
- `ChargeStart`, `ChargeRelease` and `ChargeCancel` for Chargeable Abilities, as set out in [ADR-0006](adr/0006-server-timed-charging-with-a-two-stage-cast.md)
- `CastRejected`
- `DashRequest`
- `DashRejected`
- `CastStarted`, for cosmetic replication
- `ReplaceRequest`, a Pickup and a slot only, never an Ability or Rank
- `ReturnToHub`, carrying nothing
- `PickupCollected`, which drives everyone's feedback for a Pickup taken
- `PlayerEliminated` and `MatchEnded`, the Match's feedback and results
- `CombatResult`, which drives local feedback from one authoritative result

Do not create a new RemoteEvent for each visual effect.

## Tests

Small deterministic tests are enough, without an elaborate framework. They cover pure `Rules`, such as:

- Wet and Lightning Reaction resolution
- Status consumption
- Reaction ownership and credit
- Rank progression
- Kit building, equipping, the Equip Lock and the Kit's attributes
- Fits, Upgrades and replacement, who collects a Pickup, and the checks on a replacement
- The Match's phases, roster, eliminations, end and gates, and its steps in the world on fakes
- The equal loot bag and its deal, Cache stages, drops and gates, the Sentry cycle, and what a hit does to each kind of target
- Shield absorption and lifetime, Rift Hook's pull, the generic Instant Cast path (`CastsSpec`, through `Casts.start({ manual = true })` and `Casts.receive`), and the burst limits calculated from config
- The real Match map's markers and the bays' lines of fire (`MatchMapSpec`, which needs the map in the workspace)
- The run-test telemetry: pacing milestones, central fights, guards, Shield absorbed and Hook pulls (`SessionSpec`, on a fresh `Session` with fakes)

They never run in the game by themselves. During Play, run `require(game.ServerStorage.Tests.RunAll).run()` on the Server DataModel, which returns a pass and fail summary.

## Working safely with Studio-hosted code

1. Publish the test place early, so that Roblox version history exists.
2. Before substantial MCP-driven code changes, make sure there is a known-good published or saved checkpoint.
3. Record meaningful checkpoints in the local docs when useful.
4. Avoid giant MCP edits across many unrelated ModuleScripts in one operation.
5. Prefer focused changes that can be inspected and tested before moving on.
