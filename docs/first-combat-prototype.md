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
| Tidal Bolt | Water projectile | Applies Wet and pushes lightly. |
| Arc Burst | Lightning, short-range cone | Triggers the Wet Reaction. |
| Frost Lance | Frost projectile | Two successful applications within the window cause Frozen. |
| Seismic Slam | Physical, close-range heavy AoE | Knockback and Shatter-capable. |
| Rift Wall | Space control | Temporary barrier that blocks movement and projectiles from both sides. |
| Rift Step | Mobility | Short teleport toward the Aim Point with a meaningful cooldown. |

## Basic Attack

The Basic Attack keeps players aiming, moving, applying pressure and finishing weakened opponents while Abilities are on cooldown. Staying at maximum range and spamming it should not become the optimal strategy.

| Item | Starting rule |
|---|---|
| Form | A ranged projectile with visible travel time, not hitscan |
| Firing | Hold the input to fire about once every 0.4 s, with no repeated clicking |
| Damage | About 6, with no damage falloff initially |
| Speed | About twice the projectile speed of Tidal Bolt, so it needs some leading but is considerably easier to land |
| Range | About 60 studs, after which it disappears harmlessly |
| Movement | Players move and aim normally while firing, with no slowdown, and Dash under the Dash compatibility rules |
| Presentation | Lighter than an Ability: a small muzzle or cast effect, a light trail and a lighter firing sound, with normal confirmed-hit feedback |

It uses the same authoritative projectile model as Abilities, with no separate trust model. Local firing presentation starts immediately, the server validates cadence and state and owns the hit, other clients reconstruct the visual, and damage waits for Server Confirmation. Holding the input never lets the client choose the actual fire rate.

If playtesting shows players avoid closing distance and just poke from range, tune in this order:

1. projectile speed
2. range
3. Basic Attack damage
4. Arc Burst and close-range reward
5. only then, consider damage falloff

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

- No ordinary single hit deals more than about 40% of maximum health.
- No single setup and payoff sequence deals more than about 60% of maximum health.
- Rank advantages matter, but a high-Rank player cannot instantly delete someone with a functional lower-Rank Build.

### Starting numbers

These are provisional. Do not spend much time balancing them before multiplayer testing.

| Item | Starting value |
|---|---|
| Health | 100 |
| Basic Attack | About 6 damage, about 0.4 s cadence |
| Tidal Bolt | About 12 damage, about 3 s cooldown |
| Arc Burst | About 14 base damage, with a significantly stronger Wet Reaction |
| Tidal Bolt into Arc Burst | About 40–45% of total health |
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
- how often kills happen immediately after another fight

If fights run too long mainly because players miss, prefer increasing damage or adjusting Ability cadence before making Hit Tolerance more generous.

## Dash rules

All values are provisional and configurable.

| Item | Starting rule |
|---|---|
| Duration | About 0.25 s |
| Distance | About 20 studs, purely a tuning value |
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
