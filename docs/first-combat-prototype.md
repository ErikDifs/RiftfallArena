# First combat prototype

This is the working plan for the first playable combat. Numbers are starting values to tune through playtesting, and Ability names may change.

## Goal

Prove that this loop is fun before adding more content:

**move → aim → cast → apply Wet → close distance → trigger Reaction → disengage**

## Build order

Build and test these first, then stop:

1. Basic Attack
2. Dash
3. Tidal Bolt
4. Arc Burst

The [playtest plan](playtest-plan.md) defines how this is tested. Only if that loop already feels good, add:

5. Frost Lance
6. Seismic Slam
7. Rift Wall
8. Rift Step

## Roster

The Basic Attack is untyped and non-Reactive, so that the combo rules stay easy to understand. It applies no Statuses and does not trigger Reactions or Shatter.

| Ability | Type | Behaviour |
|---|---|---|
| Tidal Bolt | Water projectile, on Q | Applies Wet and pushes lightly. Applying Wet again restarts it, and the latest character becomes its Applied By. |
| Arc Burst | Lightning, short-range flat sector, on E | Hits everyone in the sector that cover doesn't hide, and triggers the Wet Reaction. |
| Frost Lance | Frost projectile | Two successful applications within the window cause Frozen. |
| Seismic Slam | Physical, close-range heavy AoE | Knockback and Shatter-capable. |
| Rift Wall | Space control | Temporary barrier that blocks movement and projectiles from both sides. |
| Rift Step | Mobility | Short teleport toward the Aim Point with a meaningful cooldown. |

## Basic Attack

The Basic Attack is close-range filler and pressure, which keeps players moving and finishing weakened opponents while Abilities are on cooldown. Its range is short so that closing distance matters.

| Item | Starting rule, all configurable |
|---|---|
| Form | A quick forward melee swing, using the same flat horizontal sector approach as Arc Burst, so camera pitch doesn't make a swing miss |
| Range | 8 studs |
| Width | 120° |
| Height allowance | ±5 studs |
| Firing | Hold the input to swing about once every 0.4 s, with no repeated clicking |
| Damage | About 6 |
| Targets | Every valid target in the sector that cover doesn't hide, so it can cleave in free-for-all fights |
| Knockback | None, so it can't push targets out of range, interrupt Charging or act as a rapid control tool |
| Movement | Players move and aim normally while swinging, with no slowdown, and Dash under the Dash compatibility rules |
| Presentation | Lighter than an Ability: a swing effect and a lighter sound, with normal confirmed-hit feedback |

The server resolves a swing when it receives and validates it, with no wind-up. The client plays the swing immediately, and the server owns the cadence, the hit and the damage. Holding the input never lets the client choose the actual rate.

Forgiveness comes from the attack's own range and width. Hit Tolerance doesn't grow for melee. If multi-target swings prove too strong or chaotic, consider hitting only the first valid target, rather than changing the sector's geometry.

### Melee and latency

There's no rewind or lag compensation, and melee exposes that trade-off more strongly than projectiles do. Don't compensate in advance with more Hit Tolerance, client-authoritative hits or rewind. Stage 3 specifically observes whether swings near the edge of range feel unfair under real latency. If that becomes a repeated complaint, revisit the lag compensation option in [ADR-0001](adr/0001-server-authoritative-hits-with-cosmetic-client-projectiles.md), rather than hiding it with oversized tolerance.

### Melee and disengaging

Replacing a ranged Basic Attack with melee makes disengaging more valuable, which is accepted. Don't compensate yet with more Tidal Bolt uptime, faster movement, more melee range or another ranged filler attack. If fights consistently exceed the 6–9 second target, or players disengage too easily, tune from Stage 3's evidence.

## Charging

Holding an Ability's input to build a stronger Cast. It should be a real decision: cast quickly for responsiveness, or become vulnerable and predictable for a larger, easier-to-land attack. Maximum charge must not be the obviously correct choice every time.

Charging costs a player three things, but doesn't leave them helpless:

- **Action freedom:** Charging is one exclusive combat state, not separate handling for each input. While Charging, the Basic Attack is unavailable, and no other Ability can Charge or cast. Releasing casts the Charging Ability normally.
- **Mobility:** walking drops to about 75% of normal speed, 16 to 12 studs per second, with a configurable multiplier. The server owns this modifier, the client reflects it immediately enough to feel responsive, and Movement Validation uses the server's state. Normal speed returns immediately when Charging ends in any way.
- **Predictability:** opponents can see the Telegraph and respond.

| What happens while Charging | Result |
|---|---|
| Dash | Remains available, and cancels Charging |
| Knockback | Interrupts Charging, because continuing to Charge while being thrown would feel wrong |
| Stun, Frozen or death | Interrupts Charging |
| Root | Prevents movement under its normal rules, without cancelling Charging |
| Ordinary damage | Does not interrupt Charging, so rapid Basic Attacks and incidental monster hits can't suppress it |

**Any Knockback interrupts Charging, and ordinary damage does not.** There is no strength threshold, no exception for any Ability and no per-Ability interrupt logic.

This makes Tidal Bolt a deliberate Charging counter. If a player Charging Arc Burst is hit by Tidal Bolt, they take its normal damage, Wet and Knockback, and the Knockback interrupts their Charging. That's acceptable because the bolt has to land on a moving target, it has a meaningful cooldown, and the Charging player can see it coming and Dash, at the cost of their own Charge.

Skilled players may fish for these interrupts, which isn't being solved in advance. Session logs count interrupted Charging by cause. If Tidal Bolt interrupts too much in playtesting, tune the Knockback of its tap and low-charge Casts first, before adding any more complicated rule.

### Cooldowns and lost Charges

An Ability's cooldown starts only when Charging becomes a Cast, whether that's a tap, a partial Charge or a full one. The server rejects any attempt to start Charging while the Ability is on cooldown.

A Charge lost to Dash, Knockback, Stun, Frozen or death puts the Ability on no cooldown. The player loses the time invested and the opportunity they committed to, and a Dash also spends its normal Dash charge.

After a lost Charge, holding the key does nothing. The player must release it and press again, which the client enforces, while the server validates every new start independently. A fresh press can tap-cast straight away if the Ability is otherwise available. That's intentional: Tidal Bolt's reward is removing the stronger Cast and displacing the opponent, not disabling Arc Burst for several seconds.

There is no lockout, partial cooldown or minimum delay in this first version. If playtesting shows interrupts are too weak, add the smallest possible lockout then.

### Tap Window

A press released within the Tap Window, starting at 0.12 s, is an exact tap: 0% Charge and the exact tap values, with no slowdown or Telegraph. The one-action rule still applies from the press itself. This keeps quick Casts deterministic, visually clean and free of meaningless Charge caused by network timing.

Past the window, the Ability enters visible Charging, and the slowdown and Telegraph begin. Charge rises linearly from 0% at 0.12 s to 100% at 1.25 s, both configurable.

The server decides between a tap and a Charge using its own timing, and never trusts a client's Charge percentage. Timing very close to the window may fall on either side under jitter, but the Charge at stake is too small to matter. The client uses the same configured window for its presentation, so an ordinary tap doesn't flash the Telegraph or slowdown.

### Charging Telegraphs

Both Chargeable Abilities have a Telegraph that starts when the Tap Window passes, which the caster also sees.

- **Tidal Bolt:** a water orb in front of the caster, growing continuously with Charge. Its size corresponds closely to the projectile radius that would fire at that Charge, so it shows how large the bolt will be and roughly how hard it'll be to dodge. It has no ground preview.
- **Arc Burst:** lightning around the caster whose intensity grows with Charge, with a charging sound that develops over the Charge. Only the caster sees the ground preview. Others read the threat from the Telegraph, the caster's position and facing, and the full-Charge cue.
- **Full-Charge cue:** one distinct visual and one distinct audio cue, for the caster and nearby opponents, once when an Ability first reaches 100%. It doesn't repeat during the hold.

There is no HUD Charge bar in the first prototype, so players read Charging through the world. Telegraphs must not hide a player's silhouette, Status visuals, nearby attacks or aim direction, because readability outranks spectacle.

Once the Tap Window has passed, the server exposes through replicated state, using the existing tag or attribute approach, that the character is Charging, which Ability, and enough timing for clients to drive the Telegraph consistently. It clears that state when Charging ends. Clients build the progression locally, and Charge percentage is never streamed.

A remote Telegraph may lag the real Charge slightly because of latency, and that's accepted. When a Cast is replicated, its cosmetic visuals use the server's Charge value, not whatever the observing client's Telegraph had reached.

### Input rules

Combat actions are explicit rather than queued: a fresh press leads to a visible action, and nothing buffered starts later.

| Case | Result |
|---|---|
| Pressing Q or E during a Dash | Allowed, and Charging begins. Only a new Dash that starts while Charging cancels it |
| Releasing during a Dash | Casts normally, under normal server validation |
| Pressing Dash while Charging with no Dash charges | No Dash, Charging continues, and no time is lost or cancellation logged |
| A Dash the client started but the server rejects | Charging stays cancelled and is logged under `Dash`, because recreating a Charge the player has visibly lost would be more confusing. The server ends Charging on any `DashRequest`, so it knows this cause itself. The Dash itself reconciles under the normal rules |
| Holding the Basic Attack, then starting Charging | Charging takes priority and swings stop. If the input is still held when Charging ends in any way, swings resume |
| Pressing another Ability while Charging | Ignored and not queued, and the current Charge continues |
| Holding Q and E, with only Q Charging | Releasing Q follows the normal rules, and E needs a release and a fresh press |

If swings resuming after an interrupt prove unwanted, revisit that one behaviour.

### Holding and expiry

The server may decide whether a Cast is valid, but it must not invent where the player aimed.

| Item | Starting value, configurable |
|---|---|
| Maximum Charge | 1.25 s |
| Maximum hold at full Charge | 0.5 s |
| Client auto-release | About 1.75 s after Charging starts |
| Server grace | 0.5 s |
| Server expiry | About 2.25 s after Charging starts |

Reaching maximum Charge doesn't Cast. When the hold runs out, the client sends the same release as a manual one, with its current Aim Point, so auto-release has no separate trust model. The server checks that the player is still Charging, derives and clamps the Charge duration from its own time, validates the Aim Point, performs the Cast and starts the cooldown.

The server tracks the Charging state's lifetime independently. If no valid release arrives by expiry, Charging ends with no Cast and no cooldown, normal speed returns, the Telegraph ends, and a fresh press is needed to start again. A late release within the accepted window is clamped to maximum Charge. One arriving after expiry is ignored, and an attempt identifier stops it applying to a newer Charging attempt.

Session logs count `Expired` alongside the other causes. If it occurs for normal players, investigate latency and reliability, or increase the grace period.

### What Charging changes

Geometry and numbers scale continuously and linearly with the server's Charge percentage, and all are configurable. Damage multipliers apply on top of Rank values.

| Tidal Bolt | Tap | Full Charge |
|---|---:|---:|
| Projectile radius and visual size | 0.6 | 1.8 |
| Projectile speed | 80 | 65 |
| Knockback | 25 | 40 |
| Damage | 12 | 15 |

A tap is smaller and faster, so it's better for catching or interrupting someone. A full Charge is larger, easier to connect with and displaces further, but its Telegraph is more obvious. Wet doesn't change with Charge. Hit Tolerance doesn't grow either, because the larger radius is itself the extra forgiveness.

| Arc Burst | Tap | Full Charge |
|---:|---:|---:|
| Range | 18 | 28 |
| Width | 90° | 120° |
| Damage | 14 | 16.1 |
| Wet Reaction bonus | 16 | 16 |

Charging Arc Burst mainly increases its coverage and the likelihood of landing the payoff, not the payoff itself. The cooldown starts on release, so a full Charge never wins on damage over time.

If players routinely stand at range and full-Charge Arc Burst rather than engaging with positioning and Dash, tune in this order, and don't raise damage to compensate:

1. full-Charge Arc Burst range
2. width growth
3. the commitment or cost of Charging, only if necessary

### Tidal Bolt and cover

A projectile's centre path decides whether it strikes world geometry, and its radius isn't swept against cover, so large Charged bolts don't unexpectedly collide with nearby walls and corners. A big bolt may visually overlap the edge of cover if its centre path clears it.

Before any projectile hit on a character is accepted, the normal radius check must pass, and then at least one of the target's sight points, the same ones Arc Burst uses, must be visible from the projectile's current position on the server. A character fully protected behind cover isn't hit, while a genuinely exposed part of them still counts. This applies to tap and Charged bolts, dummies, and future projectiles sharing this path. Projectile radius, Hitbox size and Hit Tolerance remain separate, and Hit Tolerance doesn't change.

### Arc Burst's hit area

Arc Burst is a readable close and mid-range area Ability, not a precision anti-air attack, so it uses a flat horizontal sector rather than a 3D cone. Camera pitch doesn't tilt it. A target is hit when:

- its Hitbox falls within the sector, from the caster's horizontal aim direction, range and width
- it is within a vertical allowance of the Cast's origin, starting at ±8 studs and configurable
- line of sight and cover checks pass

Targets outside the vertical allowance aren't hit, even if the player aims directly at them. If common elevation changes make it feel unable to connect, adjust the allowance rather than returning to a 3D cone. Its cosmetic presentation stays level to match.

### Arc Burst ground preview

While Charging Arc Burst, the local player sees the same sector projected on the ground, following their aim direction and expanding with Charge from the tap geometry. Presentation raycasts make it approximately follow nearby ground, rather than float across obvious changes in height. It is presentation only, and the server decides all hits, but what it shows should closely match what the server can hit. There is no hidden 3D cone behind it.

## Casting Styles

Not every Ability is Chargeable, and an Ability's Casting Style is data-driven where practical. Charging stays a distinctive tactical mechanic, not the default for every spell.

| Casting Style | How it works | First loop |
|---|---|---|
| Instant | Press, and it happens immediately | The Basic Attack is the same in spirit, but repeats while held |
| Committed Cast | A fixed wind-up the player doesn't control, which may commit them depending on the Ability | None |
| Chargeable | Hold to Charge, and release to Cast using the server's Charge duration. Charging changes meaningful geometry or behaviour, not just raw damage | Tidal Bolt and Arc Burst |

[Future Abilities](future-abilities.md) holds candidates for each style. They don't change this milestone's scope.

## Prototype Character

Everyone uses the same controlled appearance, backed by the fixed Hitbox from [ADR-0004](adr/0004-fixed-gameplay-hitbox-independent-of-appearance.md). This removes variability from silhouette, animation, proportions, accessories, Status readability and perceived hit accuracy.

- **Identification:** players have name labels and a clear player-specific accent, which must not interfere with Status readability.
- **Status visuals:** Wet, Frozen and later Statuses are designed around this silhouette and stay understandable at combat distance.
- **Animations:** keep the scope deliberately small, and placeholders are acceptable. The set is locomotion, Basic Attack, cast, Dash, hit reaction and possibly death. These are manual or asset work, and the code integrates and triggers them. Do not build a complete set or a large animation pipeline before the first multiplayer playtest.

## Feedback

The first loop has a focused feedback layer, so that a poor playtest result reflects the combat rather than flat presentation.

| Moment | Immediately on the client | After Server Confirmation |
|---|---|---|
| Cast | Animation, sound, Cosmetic Projectile or cast VFX, and a very small camera kick where appropriate | Cancel and reconcile if the Cast is rejected |
| Hit, for the attacker | None | Hit marker, distinct hit-confirm sound, damage number for their own damage |
| Hit, for the victim | None | Hit sound, directional damage indicator, brief screen-edge feedback, small camera response |
| Reaction | None | Distinct sound, larger and more recognizable world effect, stronger attacker and victim feedback, and a differentiated damage number where useful |
| Evade | Dash and Evade visual | Clear `Evaded` response for the attacker, lightweight confirmation for the evading player, and a distinct sound or marker |
| Kill | None | Kill marker and sound for the killer, and a clear identification of the killer for the victim |
| Low health | None | A clear state on the affected player's own screen, with a configurable threshold and intensity and no excessive pulsing or flashing |

- **Private numbers:** another player's damage numbers are never shown.
- **Reactions:** a Reaction should be understandable even to someone watching the fight, but its VFX must not obscure players or make the next combat decision unreadable.
- **Camera and screen effects:** there is no real hit-pause or global time manipulation. Use small local effects instead, with configurable intensity that is easy to reduce or disable.
- **Networking:** the server sends one authoritative combat result for a hit, and clients derive their local feedback from it. Hit markers, damage numbers, sounds, camera shake and Reaction flashes do not get their own network messages unless later profiling or design gives a reason.
- **Audio priority:** the normal hit-confirm and Reaction-confirm sounds matter most for the first multiplayer test. Use placeholder or Creator Store assets rather than waiting for custom audio.

## Combat pacing

A normal fight between evenly matched players, with most attacks connecting, takes roughly 6–9 seconds. A strong Reaction sequence can shorten that substantially. A surprised player should normally have time to react, Dash, reposition or counterplay, so do not tune toward 2–4 second kills.

### Burst limits

These are design limits rather than hard engine constraints, so an exceptional future Ability can break them deliberately, not accidentally.

- No ordinary single hit deals more than about 40% of maximum health. A full Rank 3 Arc Burst Reaction landing slightly above that, at 40.7, is acceptable, but going significantly beyond it needs a new design decision.
- No single setup and payoff sequence deals more than about 60% of maximum health. A full Charge sequence at Rank 3 reaches 59.45, very close to this ceiling, so don't raise Charge damage multipliers without revisiting these limits.
- Rank advantages matter, but a high-Rank player cannot instantly delete someone with a functional lower-Rank Build.

### Starting numbers

These are provisional. Do not spend much time balancing them before multiplayer testing.

| Item | Starting value |
|---|---|
| Health | 100 |
| Basic Attack | About 6 damage, about 0.4 s cadence |
| Tidal Bolt | About 12 damage, about 3 s cooldown |
| Arc Burst | About 14 base damage, with a significantly stronger Wet Reaction |
| Tidal Bolt into Arc Burst | About 40–45% of total health at Rank 1 with tap Casts, about 42 damage. A Charged sequence may modestly exceed this, because the player paid for it |
| Rank scaling | About 20–30% total numerical improvement by Rank 3, depending on the Ability |
| Dash | About 2 charges, about 4 s recharge per charge, for roughly 2–3 meaningful Dashes per fight |

Rank 3's behaviour change should provide much of the excitement, rather than large raw damage increases.

### Recovery

| Type | Starting rule |
|---|---|
| Natural Recovery | Begins after about 6–8 s without player or monster damage, then restores about 8–10 health per second. Taking damage stops it and resets the delay. Delay and rate are configurable. |
| Kill Recovery | A player kill restores about 25% of maximum health, rewarding the win while leaving the winner vulnerable to third parties. If cleanup kills still feel excessive, tune this value before adding other mechanics. |
| Monster kills | Normal monster kills restore no health, so PvE farming does not become unlimited sustain. |

Healing monsters, Pickups or map mechanics are not being added now, but could be tested if recovery becomes too scarce.

### What to measure

Balance from measured combat, not only spreadsheet DPS. Track at least:

- average PvP fight duration
- damage dealt by Reactions versus normal attacks
- Dashes used per fight
- percentage of attacks that land
- how often players successfully disengage
- how often Natural Recovery starts
- Dash usage while chasing
- how often fights end because neither player can reconnect
- how often kills happen immediately after another fight
- how often Charging ends without a Cast, by cause: Dash, Knockback, Stun, Frozen, death, `Expired` or `ClientCancelled`

If fights run too long mainly because players miss, prefer increasing damage or adjusting Ability cadence before making Hit Tolerance more generous.

## Dash rules

All values are provisional and configurable.

| Item | Starting rule |
|---|---|
| Duration | About 0.25 s |
| Input | Left Shift |
| Distance | About 20 studs, purely a tuning value. A wall or cover ends the Dash early, and the charge stays spent. |
| Direction | Movement-input direction, or straight backwards relative to the aim direction when there is no movement input, so that Dash always works as an escape |
| Crowd control | Unavailable while Rooted, Stunned, Frozen or under any future state marked as preventing mobility. The server validates this. |
| Knockback | Interrupts an active Dash. The player may spend another available charge to recover, but no free recovery Dash is granted. |

The client may start a Dash immediately. The server validates available charge, cooldown, gameplay state and movement legality. A rejected Dash reconciles under the rules in [ADR-0002](adr/0002-client-controlled-movement-with-server-validation.md).

### Evade

Evade lasts about 0.2 s, and begins when the server accepts the Dash and consumes its charge. It does not exist just because the client visually started a Dash.

- **During Evade:** incoming hits do no damage and apply no Statuses, Knockback or other hit effects. Projectiles continue through rather than being destroyed as if they hit.
- **Evade does not:** remove existing Statuses such as Wet or Frost, or protect the player outside the accepted Dash window.
- **Readability:** Dash has a distinct visual treatment that shows Evade without obscuring the player or making aiming impossible. A hit rejected by Evade gives clear, lightweight feedback, such as an `Evaded` indicator, so the attacker doesn't read it as failed hit detection.

### Abilities and Dash

This is Ability metadata and state, not bespoke logic for individual Abilities. Each Ability defines both whether it can be used during a Dash and whether its wind-up can be cancelled by a Dash.

| Ability | Usable during Dash |
|---|---|
| Basic Attack | Allowed |
| Tidal Bolt, Arc Burst, Frost Lance | Probably allowed |
| Seismic Slam | Not freely usable, because it is committed and heavy |
| Future channelled or heavy Abilities | Generally commit the player |

Fast, light Abilities can allow a Dash during their wind-up. Committed, heavy Abilities can lock Dash temporarily as part of their cost.

### What to measure for Dash

- Dash uses and Evades per fight
- percentage of incoming hits avoided through Evade
- attacks rejected due to Evade
- deaths immediately after apparently successful Dashes
- player complaints about attacks feeling impossible to land

If Evade becomes too dominant, shorten its duration before changing Hit Tolerance. If apparently clean Dashes still result in frequent unfair-feeling hits, revisit lag compensation as described in [ADR-0001](adr/0001-server-authoritative-hits-with-cosmetic-client-projectiles.md).

## Spawn rule

A lightweight prototype rule rather than a spawn-safety system. On every spawn and respawn, the server chooses among the existing Spawn pads:

- the pad with the greatest distance to its nearest living player
- never a pad that is occupied or immediately adjacent to another player, about 12 studs, if a safe one exists
- at random between pads about as far as each other, within about 5 studs
- the farthest pad even if none is safe, rather than delaying the respawn

Do not add temporary spawn invulnerability, hidden respawn areas, dynamic spawn creation or complex line-of-sight scoring unless Stage 3 shows spawn killing is still a problem. Each choice is logged with its pad and distance, so that testing can show whether it behaves sensibly.

## Frozen rules

The goal is a strong setup opportunity without long periods where the victim cannot play.

- Frozen lasts about 1.5 seconds.
- After Frozen ends, the target has about 4 seconds of Frost Immunity.
- Shatter immediately ends Frozen.
- Ordinary incoming damage reduces the remaining Frozen duration, even if it does not trigger Shatter. The exact reduction is still to be tuned.

## Rift Step rules

Rift Step is a known movement source for Movement Validation. The client can show it immediately, and may send an Aim Point as a request, but it never supplies the destination. The server works out the destination and validates:

- cooldown and state
- maximum distance
- destination plausibility
- line of travel and obstruction rules where applicable
- that the path does not cross a Rift Wall

## Rift Wall rules

- The server has an authoritative representation used for movement and projectile checks.
- Clients have a matching representation so that Cosmetic Projectiles visibly collide with it.
- Server and clients share the same geometry and positioning contract. The wall must not depend on purely cosmetic client collision.

## Ability data

Ability metadata must let Upgrades change parameters such as damage, cooldown, projectile speed, range, push strength, Status application and Reaction modifiers, without a separate Ability implementation for each Upgrade.
