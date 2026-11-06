# Milestone 1 tickets

The ordered work for the first combat milestone: **Basic Attack → Dash → Tidal Bolt → Arc Burst → Wet Reaction**. The design lives in the [first combat prototype](first-combat-prototype.md), the [code structure](code-structure.md) and the [playtest plan](playtest-plan.md).

Work stops after each ticket until the developer gives the go-ahead. Game feel is judged by playing, so each ticket lists what only a person in Studio can check.

## Checkpoints

| Date | Checkpoint | Contents |
|---|---|---|
| 2026-11-06 | `Riftfall Arena.rbxl` at git commit `806f524` (saved, not published) | Empty baseplate, spawn and default lighting, with no scripts |

The place is not published yet (`PlaceId` 0), so Roblox version history does not exist and the saved `.rbxl` is the only code history. Code edits only reach that file when Studio saves it. Publish a private place before Ticket 2, as the first [Studio safeguard](code-structure.md#working-safely-with-studio-hosted-code) asks, rather than waiting for Ticket 8.

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

## 5. Tidal Bolt

**Scope:** the first Ability through the shared projectile path, Ability input and cooldown display, server-owned Wet with Applied By tracking, the Wet visual on the character, and the light push as Knockback under ADR-0002.

**Acceptance checks:**

- Wet appears only after Server Confirmation, and expires at its configured duration.
- The server enforces the cooldown, and a Rejected Cast restores the cooldown display.
- Knockback direction and strength come from the server, and Movement Validation allows for it.
- Evade prevents damage, Wet and Knockback, but does not remove an existing Wet.
- Adding the Ability needed data and only small generic changes, not a separate projectile path.

**Test in Studio:** hit moving dummies. Judge whether Wet is readable at combat distance without any UI, how the push feels, the bolt's speed relative to the Basic Attack, and whether the cooldown suits the fight's rhythm.

## 6. Arc Burst

**Scope:** the short-range Lightning cone, the shared Reaction system driving the Wet Reaction from `ReactionRules`, and payoff feedback for the attacker, victim and spectators.

**Acceptance checks:**

- Arc Burst uses the Hitbox for cone checks, and the server decides the hit.
- Arc Burst on a Wet target triggers the Reaction and consumes Wet, as its data says. Without Wet it deals base damage.
- Applied By and Triggered By are both tracked, and the triggering hit gets the damage and kill credit.
- The Combat code has no Wet-specific or Arc Burst-specific Reaction logic.
- The Basic Attack never triggers the Reaction.

**Test in Studio:** apply Wet, close the distance and fire Arc Burst. Judge whether the payoff feels strong, whether a watcher can tell what happened, whether the VFX hides the next decision, and whether the cone's size and range feel fair.

## 7. Fight loop

**Scope:** health, Natural Recovery, Kill Recovery, death and respawn, the remaining feedback (Kill, low health, killer identification, directional damage indicator) and session logs.

**Acceptance checks:**

- Natural Recovery starts after the configured delay, and damage stops it and resets that delay.
- A player kill restores about 25% of maximum health, credited to the triggering hit.
- Death and respawn return a player with a clean Kit and no Statuses.
- Session logs record fight duration, attack attempts, confirmed hits, Ability usage, Reactions, Dash and Evade counts, and kills and deaths.
- Low-health and screen effects have configurable intensity, and do not pulse excessively.

**Test in Studio:** fight dummies through several deaths. Judge how long a fight takes against the 6–9 second target, whether a disengage resets it, whether you can tell what killed you, and whether the low-health state helps.

## 8. Test readiness

**Scope:** fixes from a Stage 2 multi-client test in Studio, then a published private place ready for Stage 3.

**Acceptance checks:**

- The Stage 2 list in the playtest plan has been run on 2+ clients, and its problems are fixed or written down.
- What each player sees has been compared under simulated latency.
- Testers can join the published place, and session logs can be read from a real server.
- A checkpoint is recorded above.

**Test in Studio:** run the multi-client test with the developer watching each client, then join the published place from a real client. Stage 3 itself, with outside testers, is the only stage that can pass the prototype.
