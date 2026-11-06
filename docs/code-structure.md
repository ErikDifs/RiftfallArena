# Code structure

The structure serves the first combat loop: **Basic Attack → Dash → Tidal Bolt → Arc Burst → Wet Reaction**. Anything that does not help implement, test or observe that loop probably does not belong in the first milestone. Do not add `Monsters`, `Pickups`, `Match`, `Inventory` or similar until their milestone begins.

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
    │   ├── HitboxRules
    │   ├── CastRules
    │   ├── DashRules
    │   └── MovementRules
    └── Net
        └── Protocol

ServerScriptService
└── Server
    ├── Main
    ├── Characters
    ├── Dummies
    ├── Casts
    ├── Projectiles
    ├── Combat
    ├── Statuses
    ├── Movement
    └── Session

StarterPlayer
└── StarterPlayerScripts
    └── Client
        ├── Main
        ├── Input
        ├── Camera
        ├── Dash
        ├── Casting
        ├── CosmeticProjectiles
        ├── Feedback
        └── Hud

ServerStorage
└── Tests
    ├── RunAll
    └── <Name>Spec
```

The server creates a `Remotes` folder under `Shared/Net` at startup, from the events defined in `Protocol`. Avoid dots in instance names, because the MCP reads them as path separators.

## Rules for each part

- **`Config`** holds data that both client and server are allowed to know.
- **`Rules`** holds deterministic, pure logic where practical, free of Roblox object dependencies.
- **`Main`** on each side only requires and starts systems, provides explicit dependencies and establishes startup order. It does not accumulate gameplay logic.
- **Shared code** does not hold security-sensitive decisions. The server remains authoritative over outcomes.

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
- `CastRejected`
- `DashRequest`
- `DashRejected`
- `CastStarted`, for cosmetic replication
- `CombatResult`, which drives local feedback from one authoritative result

Do not create a new RemoteEvent for each visual effect.

## Tests

Small deterministic tests are enough, without an elaborate framework. They cover pure `Rules`, such as:

- Wet and Lightning Reaction resolution
- Status consumption
- Reaction ownership and credit
- Rank progression

They never run in the game by themselves. During Play, run `require(game.ServerStorage.Tests.RunAll).run()` on the Server DataModel, which returns a pass and fail summary.

## Working safely with Studio-hosted code

1. Publish the test place early, so that Roblox version history exists.
2. Before substantial MCP-driven code changes, make sure there is a known-good published or saved checkpoint.
3. Record meaningful checkpoints in the local docs when useful.
4. Avoid giant MCP edits across many unrelated ModuleScripts in one operation.
5. Prefer focused changes that can be inspected and tested before moving on.
