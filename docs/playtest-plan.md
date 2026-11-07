# Playtest plan

This plan tests the first loop in the [first combat prototype](first-combat-prototype.md): Basic Attack, Dash, Tidal Bolt and Arc Burst.

## Stages

Only Stage 3 can pass the prototype. Stages 1 and 2 can show that it works technically, but not that PvP is fun.

### Stage 1: Solo in Studio

Use a static target dummy, a strafing dummy, a dummy capable of occasional Dash and Evade, a dummy that shoots back so that a solo player can be hurt and killed, and simulated latency where useful.

Validate aiming, projectile feel, cast responsiveness, hit feedback, Wet readability, Arc Burst Reaction readability, Dash and Evade timing, and obvious networking bugs.

The shooting dummy fires tap Tidal Bolts through the normal combat path, at their normal cooldown. Solo testing can therefore cover taking projectile damage, Wet, Knockback, Charging being interrupted by Knockback, death and respawn, and cover. It doesn't Charge or chase, so specs and Stage 2 cover ordinary damage not interrupting Charging, and how another player's Telegraph looks. Don't raise its fire rate or damage to kill faster: use a dedicated test setup if a quick death is needed.

### Stage 2: Local multi-client

Use Roblox Studio multi-client testing to validate client and server authority, Cosmetic Projectiles, Server Confirmation, Rejected Cast behaviour, Dash reconciliation, Evade, Status and Reaction replication, and what each player sees when latency or state differs.

### Stage 3: Real multiplayer

- 2–4 other people plus the developer, where practical
- a published, private test place
- several short sessions of about 15–30 minutes, rather than one long session
- testers do not need to be experienced Roblox players, and at least some should not have been involved in designing the combat

## Test arena

A very small greybox arena, with no meaningful art time spent on it:

- about 100 studs across as a starting point
- some line-of-sight cover
- one raised area
- one meaningful ledge or drop
- enough open space for projectiles, and enough cover for disengaging

## Most important signal: replay desire

After a fight or session ends, observe whether players naturally want to fight again, swap sides or Kits, try the combo again, beat someone who just killed them, or experiment with Dash timing. Do not ask "was that fun?" before observing what they want to do. People voluntarily asking for another round is much stronger evidence than any numeric metric.

## Directional targets

These are goals rather than pass or fail thresholds, because a handful of testers is too small a sample for percentages.

- fight duration near the 6–9 second target
- attack hit rate of roughly 35–60%
- Wet into Lightning Reactions occurring regularly
- players occasionally disengaging successfully
- Evade mattering without constantly triggering
- few obvious cases of attacks appearing broken or unfair

For the Reaction, these questions matter more at this sample size:

- Did players discover and use it naturally?
- Did it happen repeatedly during the session?
- Did players deliberately try to set it up once they understood it?
- Did opponents recognize what happened?

## Comments to record

After sessions, record comments like these, which show whether the networking and interaction model feels understandable:

- "I definitely dodged that."
- "Why didn't that hit?"
- "I don't know what killed me."
- "I couldn't do anything."
- "That Dash felt really good."
- "I was trying to make them Wet so I could Arc Burst."

## Instrumentation

Server and session logs are enough, and no telemetry backend is needed. Record fight duration, attack attempts, confirmed hits, Ability usage, Reactions, Dash count, Evade count, kills and deaths, and Charging that ends without a Cast by cause (Dash, Knockback, Stun, Frozen, death, `Expired` or `ClientCancelled`). The first prototype doc lists the fuller metrics to derive from these.

Lines starting `[Riftfall][Session]` in the server console hold these: one for each spawn, with its pad and distance to the nearest player, one for each kill, non-combat death and fight, and a JSON summary per player when they leave or the server closes. A player's fight starts with the first damage they deal or take, and ends when someone dies or after 7 seconds with no damage either way. A lone hit that nothing followed isn't counted as a fight. A hit rate counts Casts that landed against Casts accepted, so a Sector hitting two targets can't push it past 100%.

## If the loop does not feel good

Do not add Frost Lance, Seismic Slam, Rift Wall, Rift Step, monsters, progression or match structure to try to fix it. Iterate first on:

- movement
- aiming
- projectile speed
- damage and cooldowns
- Reaction payoff
- feedback, VFX and audio
- Dash and Evade
- arena layout

Do a few meaningful iteration rounds. If people still do not enjoy simply fighting each other with Basic Attack, Dash, Tidal Bolt and Arc Burst after that, stop and reconsider the combat foundation rather than building more content on top of it.
