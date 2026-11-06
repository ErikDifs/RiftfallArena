# Playtest plan

This plan tests the first loop in the [first combat prototype](first-combat-prototype.md): Basic Attack, Dash, Tidal Bolt and Arc Burst.

## Stages

Only Stage 3 can pass the prototype. Stages 1 and 2 can show that it works technically, but not that PvP is fun.

### Stage 1: Solo in Studio

Use a static target dummy, a strafing dummy, a dummy capable of occasional Dash and Evade, and simulated latency where useful.

Validate aiming, projectile feel, cast responsiveness, hit feedback, Wet readability, Arc Burst Reaction readability, Dash and Evade timing, and obvious networking bugs.

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

Server and session logs are enough, and no telemetry backend is needed. Record fight duration, attack attempts, confirmed hits, Ability usage, Reactions, Dash count, Evade count, and kills and deaths. The first prototype doc lists the fuller metrics to derive from these.

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
