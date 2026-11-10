# Code structure

The structure serves the first combat loop: **Basic Attack → Dash → Tidal Bolt → Arc Burst → Wet Reaction**. Anything that does not help implement, test or observe that loop probably does not belong in the first milestone.

[Milestone 2](milestone-2-tickets.md) adds the run progression loop, so systems for Kits, Pickups, the Match, Rift Caches and Sentries now belong, each added by its ticket. A general monster system and an inventory still don't. [Ticket 25](milestone-2-tickets.md#25-match-milestones) (Milestones, staged and not built) would add a pure `MilestoneRules`, a server `Milestones` module and a shared combat-idle signal, with Riposte and Catch Breath rules beside them. Its gated Stage 3 Reserve is **one extra field on the Kit**, not an inventory.

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
    │   ├── LootRules
    │   ├── CacheRules
    │   ├── SentryRules
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
    ├── Caches
    ├── Sentries
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
        ├── CosmeticProjectiles
        ├── CosmeticSectors
        ├── PickupVisuals
        ├── PveVisuals
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
└── Tests
    ├── RunAll
    └── <Name>Spec
```

The server creates a `Remotes` folder under `Shared/Net` at startup, from the events defined in `Protocol`, and a `Pickups` folder in the workspace for the Pickups' Parts. A Match's first setup creates `Caches` and `Sentries` folders for their models, so they never exist with the Reference Kit. Each client draws Pickups into its own `PickupVisuals` folder, and Cache and Sentry effects into `PveVisuals`. Avoid dots in instance names, because the MCP reads them as path separators.

The workspace holds three static environment folders:

- `Workspace.Arena`: the Reference Kit's arena, used for Stage 3, with pads `Spawn1`–`Spawn4`.
- `Workspace.MatchMap`: the Match's greybox map, about 250 × 250 and centred at (0, 0, −450). It has six pads, `MapSpawn1`–`MapSpawn6`, on a ring of r = 105, and three bays around a solid core. Its `Markers` folder holds the invisible, non-colliding Cache markers (`CacheOuter01`–`10` and `CacheCentral1`–`3`, each with a `CacheKind` attribute) and the Sentry markers `Sentry1`–`3`.
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

## Rift Caches and Sentries

From [Ticket 18](milestone-2-tickets.md#18-match-map-caches-and-sentries), the Match's PvE is breakable Rift Caches and stationary Sentries, owned by the server. Only `Match` sets them up and clears them, so with the Reference Kit none exist and no loop runs.

- **`LootRules`** is the equal Ability bag. It covers:
  - `validate`, which checks the pool divides the drops evenly
  - `buildBag`
  - `nearestPairs`, each pad's two nearest outer Caches
  - `deal`: central Caches first with different Abilities each, then the outer ones, then a bounded one-swap repair for early variety

  Equal counts are strict, and variety is a best effort, reported as `variety=ok` or `partial`.
- **`CacheRules`** holds the crack stage for a health, where drops go, and `mayDamage`: only a player who is Alive in a Running Match, checked at impact.
- **`SentryRules`** is one Sentry's Idle → Telegraph → Recover cycle as plain data. The aim locks when the Telegraph starts, and the shot is cancelled if the target becomes invalid or out of sight. Sight is passed in.
- **`Caches`** (server):
  - `prepare(random)` deals a bag without changing the world.
  - `setup(deal, matchId)` builds the 13 Caches from the markers. Each is a model with a hidden Humanoid, a root for the fixed Hitbox and a body sized to it, registered as a `cache` target.
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
