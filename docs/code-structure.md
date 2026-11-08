# Code structure

The structure serves the first combat loop: **Basic Attack → Dash → Tidal Bolt → Arc Burst → Wet Reaction**. Anything that does not help implement, test or observe that loop probably does not belong in the first milestone.

[Milestone 2](milestone-2-tickets.md) adds the run progression loop, so systems for Kits, Pickups, the Match, Rift Caches and Sentries now belong, each added by its ticket. A general monster system and an inventory still don't.

## Layout

```text
ReplicatedStorage
└── Shared
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
        ├── Replacing
        ├── StatusVisuals
        ├── ReactionVisuals
        ├── Feedback
        ├── HealthDisplay
        └── Hud

ServerStorage
├── PickupPlacements (hand-placed test markers, used only with an Empty Starting Kit)
└── Tests
    ├── RunAll
    └── <Name>Spec
```

The server creates a `Remotes` folder under `Shared/Net` at startup, from the events defined in `Protocol`, and a `Pickups` folder in the workspace for the Pickups' Parts. Each client draws Pickups into its own `PickupVisuals` folder. Avoid dots in instance names, because the MCP reads them as path separators.

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
- **`Pickups`** (server) keeps each Pickup's record and gives it an invisible, tagged Part in `workspace.Pickups` carrying `PickupId`, `PickupAbility`, `PickupRank` and `PickupSpawnedAt`. About 10 times a second it gives each Pickup, in ascending id order, to the nearest living, unblocked player it fits, through `Kits.collect`. It handles `ReplaceRequest` in one synchronous step, so the first valid request wins, and drops the replaced Ability at the player's feet as a new Pickup. Either way it fires `PickupCollected` and removes the Part. With an Empty Starting Kit it turns the markers in `ServerStorage.PickupPlacements` into Pickups at startup.
- **Clients** draw Pickups in `PickupVisuals`, from the tag and attributes only. `Replacing` keeps its own copy of the Kit, offers the nearest Pickup a full Kit could take only by replacing, shows the prompt, and sends `ReplaceRequest` when the player holds the modifier and presses a slot. `Input` sends a slot's press to `Replacing` instead of `Casting` while the modifier is held. Nothing on the client predicts a Kit change.

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
- `PickupCollected`, which drives everyone's feedback for a Pickup taken
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

They never run in the game by themselves. During Play, run `require(game.ServerStorage.Tests.RunAll).run()` on the Server DataModel, which returns a pass and fail summary.

## Working safely with Studio-hosted code

1. Publish the test place early, so that Roblox version history exists.
2. Before substantial MCP-driven code changes, make sure there is a known-good published or saved checkpoint.
3. Record meaningful checkpoints in the local docs when useful.
4. Avoid giant MCP edits across many unrelated ModuleScripts in one operation.
5. Prefer focused changes that can be inspected and tested before moving on.
