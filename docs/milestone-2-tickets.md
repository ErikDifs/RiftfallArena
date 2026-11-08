# Milestone 2 tickets

The ordered work for the second milestone: the **run progression loop**, **explore and fight PvE → find Abilities → change the Kit → fight players with it**. It tests whether Riftfall gets more interesting when a player's Kit develops during a run, not whether more content can be added.

Being settled from 2026-11-08, before any building. Work stops after each ticket until the developer gives the go-ahead, as in [Milestone 1](milestone-1-tickets.md).

## Settled for the whole milestone

**Stage 3 and the run test are separate gates (2026-11-08).**

- Building this milestone goes ahead now, because it can be built solo. Stage 3 never depends on it.
- The **Reference Kit** stays the default Starting Kit, so the published Stage 3 build behaves as it does today. The progression work sits behind the smallest clean switch between the Reference Kit and an empty Starting Kit, with no larger mode framework.
- The first sitting with outside testers is Stage 3 only: the current arena, the Reference Kit and the current combat loop, for about 15–30 minutes, with no need to touch the new progression. It answers whether the combat itself is fun, readable and worth continuing.
- The run test comes in a later, separate sitting, only after Stage 3's combat holds up. It answers a different question: does exploring and PvE → finding Abilities → changing the Kit → PvP make the game better?
- If Stage 3 shows the combat needs iteration, combat fixes come before any further progression or content work. This milestone isn't a reason to avoid fixing a weak combat loop.

**Two Ability Slots, on Q and E, as a milestone constraint (2026-11-08).**

- Milestone 2 uses two configurable Ability Slots, Q and E, to maximise pressure on the Kit and make replacement meaningful. This isn't a permanent cap.
- Two slots fit this milestone because there are only four Abilities by its end, so replacement happens often enough to test. Carrying Tidal Bolt and Arc Burst takes the whole Kit, so finding a third interesting Ability forces a real decision. Q and E already exist, so no new input or HUD work is needed.
- The slot count and key bindings stay in config, so a third slot on R can be tested later. R isn't built in this milestone.
- The long-term number is unresolved, with three slots a likely test. Once the pool reaches about 6–8 distinct Abilities, test two slots against three in play, rather than deciding from theory.

**Upgrades from Rank 1 to Rank 2 only (2026-11-08).** Milestone 2 includes Ability Upgrades from Rank 1 to Rank 2. Rank 3 and behaviour-changing Upgrades stay outside the milestone, because Rank 3's behaviour changes would add content and design work to every Ability.

- All normal world loot spawns at Rank 1. Collecting a duplicate upgrades the owned Ability to Rank 2.
- The maximum Rank is a config value, set to 2, so Rank 3 can return later without redesigning the system.
- Ranked loot from Elites and events waits until those systems exist.
- Ticket 18's two new Abilities get sensible Rank 2 values, and no Rank 3 behaviour.
- No upgrade currencies, trees or modifiers.
- The purpose: once Q and E are full, duplicates still give a reason to explore and contest loot. Keeping everything at Rank 1 would make duplicates dead loot, and would need a stand-in for the death rule's Rank loss.

**Rank authority (checked in the code 2026-11-08).** The server owns each equipped Ability and its Rank, and only that Rank is used for gameplay.

- No client message carries a Rank. `CastRequest`, `ChargeStart` and `ChargeRelease` carry ids, the Ability and the Aim Point, and `Protocol` drops any payload with extra fields, so a forged `rank` is refused as malformed.
- `Casts.fromKit` resolves the Rank for every request path, and reads the server-owned Kit from Ticket 15 on.
- Presentation Rank goes from the server to clients only: in `CastStarted` already, and from Ticket 15 for other players' Charging Telegraphs too. Today those read the shared constant (`ChargeTelegraphs`, line 176), because the Charging attributes carry no Rank. Clients use it for the HUD, Telegraphs and cosmetic drawing, never for outcomes.

**Fitting Pickups are collected automatically (2026-11-08).** A Pickup fits when the player doesn't own its Ability and has a free slot, or owns it below the Rank cap, so collecting it is an Upgrade. Collection can only help the player then, so it needs no input.

- The server checks living players against Pickups about 10 times a second, by the server's own position for the character, never `Touched`. A player within about 4 studs collects it, and a new Ability goes into the first free slot, Q then E.
- The first check that finds someone in range gives it to them and removes it in the same step. If several players are in range on that check, the nearest one gets it.
- Dead, Stunned and Frozen players can't collect. Rooted and Charging players can.
- An Upgrade applies to the next Cast the server resolves. A running Charge takes its Rank at release, where `fromKit` runs.
- A Pickup of an Ability the player already holds at the Rank cap stays in the world.
- Collecting is readable to others: a short world flash and sound, plus a pulse on the collector's HUD slot.

**Full-Kit replacement: hold F, press Q or E (2026-11-08).** Q and E are Chargeable, so holding or pressing them alone can't mean anything but Casting.

- **Input.**
  - Plain Q and E always Cast. While F is held, they become replacement choices, and releasing F returns them to Casting at once, with no lingering mode.
  - F alone does nothing, and holding it doesn't suppress movement, the camera, the Basic Attack or Dash.
  - With no eligible Pickup targeted, F with Q or E replaces nothing.
  - F is Milestone 2's replacement modifier, not a settled part of the final controls.
- **Target and prompt.** The nearest eligible Pickup in range is the target. The prompt shows the local player both sides of the choice, for example `F + Q — Replace Tidal Bolt with Stone Guard` and `F + E — Replace Arc Burst with Stone Guard`.
- **Authority.** The client sends `ReplaceRequest { pickupId, slot }` and doesn't predict the Kit change.
  - The server checks that the player is alive, not Stunned, Frozen or Charging, and that its own position for the character is within about 6 studs, against about 4 for automatic collection.
  - It also checks that the Pickup still exists and is the expected Ability that doesn't fit, and that the slot exists and can be replaced.
  - The first valid request wins.
  - If the Pickup has gone or the player has moved out of range, the request is refused. The Kit stays as it was, with no cooldown or Equip Lock.
- **Charging.** A swap while Charging is refused. Replacement isn't a cause that ends a Charge.
- **The replaced Ability** drops at the player's feet as a Pickup at its current Rank. The player can't collect it back automatically while their Kit is full, but another nearby player can, which makes swapping mid-fight a visible risk. There's no intentional dropping or swapping between Q and E in this milestone.
- **Cooldowns** stay keyed per Ability per life, so swapping an Ability out and back in never resets its cooldown.
- **Equip Lock.** An Ability newly equipped by replacement or automatic collection into an empty slot can't be Cast for a short, server-owned time, about 0.75–1.0 s to start with, as tuning. It stops "Cast the old Ability → swap → Cast the fresh one" from getting around the combat's pacing and the sequence damage limits. An Upgrade of an Ability already equipped doesn't lock it.

**One life is the Arena's direction (2026-11-08).** It applies to the full Match too, not just this milestone, as [ADR-0007](adr/0007-one-life-arena-match.md) records with the superseded respawn model. The full-Match docs and the glossary are updated to match.

**A cut-down, one-life Match when the Starting Kit is Empty (2026-11-08, revised the same day).** It tests empty Kit → exploring and loot → a developing Kit → PvP. It's a deliberately cut-down version of the full [Match](match-structure.md), not a separate concept. The first version, a timed Match with respawns throughout, was replaced the same day by the developer's one-life rule below.

- At the start of each Match, every player's Kit is empty and the world's Pickups are placed fresh. There are no Gather, Clash or Rift Collapse phases.
- **One life.** Death eliminates the player from the Match. They don't respawn into it, and they go into Spectate. Death means the current run is over.
- **The last living player wins.** A configurable maximum duration, about 8 minutes to start with, stays as a failsafe.
- **The Match controls participation explicitly,** as Alive, Eliminated and Spectating, or leaving for the Hub, rather than relying on Roblox's respawning. Respawning belongs to the Hub and other places outside a Match, and never returns an eliminated player to the Match.
- **Spectate** is deliberately lightweight: following the living players, cycling between them, a clear sign that the player is eliminated, and **Return to Hub**. An eliminated player can't affect gameplay, collect Pickups, Cast, move their old character or count as alive for the win.
- **Return to Hub** is available at any time while spectating, without waiting for the Match to end. A player who stays to the end goes back to the Hub through the results. Until the Hub exists, it uses the smallest temporary destination that keeps that state change clean. No matchmaking, ranked queues or social Hub in this milestone.
- **Results** at the end show each player's kills, Pickups collected, Upgrades and replacements, plus Reactions, since the session logs already count them. They're for testing, not the final scoreboard, and need no larger stats system.
- **With the Reference Kit**, the Stage 3 arena behaves as it does now: repeated respawns, no Match, no elimination. Stage 3 and the run test stay separate experiments.

**An Eliminated player drops their whole Kit (2026-11-08).**

- Each equipped Ability drops as its own Pickup at its current Rank, where the player died.
- Any death counts: to a player, a monster or a fall. A player who leaves mid-Match is Eliminated and drops the same way, so leaving can't deny the loot.
- Kill Recovery stays at 25%, since the winner of a fight often faces a third player at once.
- Dropped Pickups last until the Match ends.
- If snowballing proves too strong, keep dropping the whole Kit, but drop it at Rank 1, before changing anything else.

**Match start and the temporary Hub (2026-11-08).**

- **The temporary Hub** is a lobby area in the same server, away from the Match map. Players load there, respawn there normally, and arrive there after Return to Hub and after the results. There's no combat in it: no Casts, no Basic Attack and no damage.
- **Starting.** Once at least the minimum number of players are in the Hub, a countdown of about 15 s runs. Everyone in the Hub then enters together, at separate spawn points across the map.
- **Minimum players** is a config value: 2 normally, and 1 for solo testing in Studio.
- **No late joining.** Players who arrive after a Match starts stay in the Hub until the next Match, and can Spectate if they want to. This replaces the earlier "late joiners start with an empty Kit".
- **Only one active Match, as a Milestone 2 implementation and testing constraint, not the target architecture.** The target Hub batches players into several concurrent Matches, as [Match structure](match-structure.md#future-outer-loop) sets out. Hub matchmaking, `TeleportService` and reserved servers wait until later.
- **The Reference Kit** has no Hub and no Match.

**Match end and timeout (2026-11-08).**

- If exactly one player is left alive before the maximum duration, about 8 minutes and configurable, that player wins.
- If the time runs out with two or more alive, the Match ends with no winner. Nothing else picks one: not kills, damage, Rank, loot collected or any other secondary score, because those would reward something other than being the last player standing.
- The results show **Time out — no winner** and list the survivors.
- The session log records the end cause (`Timeout`, or a winner), the number still alive, the Match duration and the number of eliminations. The key test metric is how often Matches fail to reach one survivor naturally.
- No Rift Collapse in this milestone. Convergence comes from a small enough map, loot placement, contested valuable areas, the noise of fighting, and Eliminated players' whole-Kit drops worth contesting.
- If timeouts are more than occasional in testing, the next convergence feature is probably the smallest useful version of the Rift Collapse's shrinking playable area, not an unrelated mechanic against hiding. Evidence decides whether that ticket is needed.

**PvE is breakable Rift Caches and stationary Sentries (2026-11-08).** It's the smallest PvE that can prove the run loop, and none of the larger monster system in [Monsters](monsters.md) is built for it. It exists to test that:

- exploring has a cost, and getting loot exposes the player
- PvE can leave a Status that matters in combat, which another player can use. The ideal case: a Sentry's Tidal Bolt makes a player Wet, another player arrives, and Arc Burst triggers the Reaction.
- valuable loot creates contested places
- players have reasons to move through the map, not just hunt each other

**Rift Caches.**

- Server-owned breakable world objects with modest health, damaged by normal attacks, the Basic Attack included. Breaking one with melee alone exposes the player for a few seconds.
- A broken Cache drops one Rank 1 Ability Pickup from its configured Drop Table, with readable visual and audio feedback that nearby players can notice. It stays broken for the rest of the Match.
- No rarity, complex loot tables or Caches that come back.

**Sentries,** the only hostile PvE in this milestone.

- Built on the ShootingDummy's behaviour and path: stationary, targeting the nearest visible living Match player, firing tap Tidal Bolts through the normal authoritative Cast path.
- Its current tuning is the starting point: about 12 damage every 3 s, Wet and the normal light Knockback.
- Killable, and once killed it stays dead for the rest of the Match.
- No movement, chasing, pathfinding, Home Territory, Threat, Monster Families, Elites, scheduler, or guaranteed-first-kill rule.

**Placement is a risk gradient.** Safer, lower-value outer Caches have no Sentry. More valuable or strategically placed central Caches are guarded by Sentries. Easier progression lies around the edge, and better chances with more PvE exposure and PvP risk lie in the centre, which helps players converge without the Rift Collapse.

**One-life tuning.** PvE should matter without often ending runs by itself. A careless player can die to a Sentry, but normally PvE costs health, time and position, and leaves a Status. Health, cooldowns and Cache durability are all tuning values.

**Loot, map and pacing (2026-11-08).**

- **Pacing targets,** for tuning rather than rules: the first Ability within about 30–45 s, a full Kit within about 1.5–2 minutes, then Upgrades and replacements as the main loot decisions. They matter more than any exact map size or Cache count.
- **A new greybox Match map** of roughly 250 × 250 studs, built mainly for about 4 players, with 2–6 still workable. Spawn points sit around the edge, and the centre is meant to draw contest and convergence. The size is a starting value: if the pacing targets are missed, change the map's size and layout rather than protecting its dimensions. The current arena stays unchanged for the Reference Kit and Stage 3.
- **Outer Caches:** about 10, unguarded, roughly two within reach of each spawn area. Each drops one Rank 1 Pickup, for early, lower-risk Kit building.
- **Central Caches:** about 3, guarded by Sentries. Each drops two different Rank 1 Pickups, so they're worth more through more loot and choice, not Rank.
- **Two drops are set apart.** Because fitting Pickups are collected on proximity, a central Cache's two Pickups appear visibly apart around it, about 6–8 studs to start with, configurable. The player chooses by walking to one, and can collect both if they have room. No selection menu.
- **A shuffled bag, not pure weighted randomness.** For 16 Pickups from 4 Abilities, the bag holds 4 copies of each. It's shuffled at Match start and dealt into the Caches. Every Match then has every Ability in equal numbers in different places, so a run can't lose an important Ability to chance. The loot data stays generic enough for weighted Drop Tables to come back when the pool is bigger.
- **Early variety.** Where practical, the two outer Caches nearest a spawn area hold different Abilities, so a player's first two Caches don't both give the same Ability. A simple fixed adjustment to the dealing is enough. No general constraint solver.
- **Hidden contents.** What each Cache holds is decided at Match start but hidden until it breaks, and all unopened Caches look the same. Players choose where to go by position, risk, contest and Sentries, not by spotting a wanted Ability from afar.
- **No loose Pickups at start.** Abilities enter the world only from broken Caches, replacements and Eliminated players' Kits.
- **Questions for the run test:**
  - how long until a player's first Ability, and until two are equipped?
  - how often do players reach the centre, and how often do central Caches start fights?
  - do duplicates naturally lead to Upgrades?
  - are loot decisions still meaningful with a full Kit?
  - does 250 × 250 give enough encounters without a Rift Collapse?

**Readability (2026-11-08).** Greybox geometry, text and the existing Ability colours are enough for this milestone.

- **Pickups** can be identified without hovering or a menu. Each has:
  - the Ability's colour and a simple shape of its own, so colour isn't the only cue
  - its name, readable at about 40 studs to start with
  - one or two Rank pips, plus a brighter ring at Rank 2
- **Long-range signal.** A thin vertical beam in the Ability's colour, visible from a distance when line of sight allows, but not necessarily from anywhere on the map. It starts subtle, so many Pickups don't add up to noise, and its height, thickness, brightness and visible distance are tuning values. An Eliminated player's dropped Kit flashes or pulses strongly when it appears, then settles into the normal signal, so nearby players notice a fight has just ended.
- **Close cues, for the local player only.** An Upgrade shows `Upgrade → Rank 2`. A full Kit shows the replacement prompts. Ordinary collection into a free slot gets only a subtle cue, not dominant instruction text, since it's quickly learned by doing it.
- **Caches** all look the same unopened and never reveal their contents. Visible crack stages as they take damage, and a clear, loud break effect, let others tell that someone is opening one or just has.
- **Sentry Telegraph.** About 0.5 s of energy building at the firing point before every shot, reusing the Charging visual language where practical, within the same roughly 3 s cadence. It's readability only, not a real Charge: no hidden Charge scaling and no different projectile values. The bolt still goes through the normal authoritative Tidal Bolt path. A careful player gets a readable chance to respond before a one-life PvE attack fires.
- **HUD.** Q and E show the equipped Ability's name and colour, its Rank pips, and a clear empty state. The Equip Lock has its own look, distinct from a cooldown sweep, so a player can tell "cooling down" apart from "just equipped".
- **Out of scope:** final icons and an asset pipeline, minimap loot markers, detailed tooltips, inventory UI, elaborate rarity effects.

**The two new Abilities: Rift Hook and Stone Guard (2026-11-08).** Ticket 18 isn't just two more Abilities: it tests whether finding different pairs changes how players approach a run and a fight. Both use the existing Instant Casting Style. No new Charging Ability and no new Status chain. All numbers are Ticket 18's starting tuning.

With two slots, the four Abilities make six different Kits:

| Kit | Plays as |
|---|---|
| Tidal Bolt + Arc Burst | Setup and payoff |
| Rift Hook + Arc Burst | Aggressive engage |
| Tidal Bolt + Rift Hook | Ranged control |
| Arc Burst + Stone Guard | Durable brawler |
| Tidal Bolt + Stone Guard | Poke and survive |
| Rift Hook + Stone Guard | Close-range bruiser |

Sentry Wet keeps Statuses and Reactions relevant to Kits without Tidal Bolt.

**Rift Hook**

- A fast, narrow, untyped projectile, with a range of about 30 studs, about 8 damage and a cooldown of about 7 s.
- On a hit, it pulls the target toward a point about 4 studs in front of the caster.
- **The pull is Knockback,** now meaning forced displacement toward or away from its source, with no separate pull state. It follows the existing rules: the server decides the movement, it ends a Charge, it gets Movement Validation's allowance, and it uses the same forced-movement path.
- **Geometry safeguards.**
  - The point in front of the caster is a target, not a teleport. The pull stops short when walls or other geometry are in the way, and never goes through them.
  - It stays mainly horizontal, with no big vertical launches from height differences.
  - It's never a teleport.
- **Rank 2** improves one understandable property. Evaluate more range first, rather than raising range, damage and cooldown together.
- **To watch:** Hook → Arc Burst is meant to be a strong engage, and isn't weakened from theory. The first balance question is whether a narrow, dodgeable Hook gives the target enough chance to avoid the setup.

**Stone Guard**

- Grants a Shield of about 25 for about 3 s, with a cooldown of about 12 s, clearly visible on the character.
- **Survive the hit, not avoid it.** The Shield takes damage before health. A hit still has its normal consequences: a Tidal Bolt into Stone Guard damages the Shield, makes the target Wet and pushes them. Reactions still trigger, and their damage comes off the Shield first. No immunity to Statuses, Knockback or Reactions. Evade avoids the hit, and Stone Guard survives it.
- **Shield rules.** Shields from Stone Guard don't stack, and whatever's left disappears when it expires. Absorption goes through the existing authoritative combat path. Opponents can clearly read that a player is Shielded.
- **Rank 2** gives a stronger Shield, about 35, not also a longer duration or shorter cooldown.

**No non-Ability world items in Milestone 2 (2026-11-08).** The run test already has enough new variables: an empty Starting Kit, finding Abilities, replacement, Upgrades, Rift Caches, Sentries, dropped Kits, one life, Spectate and Return to Hub, and six possible Kits. Temporary items would make it harder to tell whether the Kit loop itself works.

- **Revisit after the first run test** if the evidence shows a specific need, such as:
  - players stop exploring once their Kit is full
  - Natural Recovery is too scarce for one life
  - the world needs more temporary tactical decisions
  - full Kits need something to contest besides Upgrades and dropped Kits

  No item system is built in advance.
- **Principle: Abilities change your Kit, and items temporarily change the situation.** No equipment slots, inventory grid, consumable hotbar, permanent stat gear, or rarity and affix systems, unless later evidence gives a strong reason.
- **Principle: Reactions stay earned.** An item may strengthen a Reaction's payoff, change what follows a valid Reaction, or extend one of its consequences. It never bypasses the Status setup, so **Status → matching effect → Reaction** stays the readable combat language. Something like Volatile Ember, which strengthens the next valid Reaction, fits. Something like Storm Core, where Lightning acts as if Wet were already there, doesn't.
- **Candidate notes, kept for later:**
  - a health pickup waits for the run test to show whether Natural Recovery is too scarce
  - a temporary Shield overlaps Stone Guard
  - a Frost item waits for Frost and Frozen
  - restoring cooldowns would work, but is generic
  - modifying a Reaction's payoff could be interesting later
- **Terms.** Pickup still means an Ability Pickup only. If items are designed, they get their own term then.

## Ticket sequence

Settled 2026-11-08. Each ticket can be tested before the next, and the Reference Kit keeps working through all of them. Pure logic goes into new `Rules` with specs (`KitRules`, `LootRules`, `MatchRules`, Rift Hook's forced displacement and geometry, and the Shield's absorption), with raycasts and other world checks passed in, as `CastRules.originFor` does, so the Rules stay testable outside Studio. `studio-tester` runs each ticket's Play checks, as in Milestone 1.

### 15. Per-player Kit

- **Scope:**
  - The server owns each player's Kit: Q and E slots, each holding an Ability and its Rank. Keys move from the Abilities in `Config` to the slots.
  - The Kit replicates, and the HUD is built from it, with name, colour, Rank pips and an empty state.
  - A presentation Rank for other players' Charging Telegraphs.
  - The Rank cap in config, set to 2. Cooldowns per Ability per life. The Equip Lock, with its own HUD look.
  - The Starting Kit setting, Reference by default.
  - A server-side grant for specs.
  - `code-structure.md` gains the new systems.
- **Authority:** the server-owned Kit is the only source for the equipped slot, Ability, Rank and whether a Cast is allowed. Any Rank sent to clients, for Telegraphs or anything else, is presentation only. No client-supplied Rank ever decides damage, geometry, cooldowns or any other gameplay.
- **Done when:** Stage 1 and 2 checks with the Reference Kit behave exactly as version 46 does.

### 16. Pickups and replacement

- **Scope:** server-owned Ability Pickups, collection by proximity, Upgrades, F+Q and F+E replacement through `ReplaceRequest`, the replaced Ability dropping, and readability in the world and HUD.
- **Where the logic lives:** fit, Upgrade and replacement decisions go in `KitRules` where practical. World proximity, instance ownership and replication stay out of the pure Rules.
- **Tested with** Pickups placed by hand in the current arena and an Empty Starting Kit.

### 17. One-life Match

- **Scope:**
  - The temporary Hub, the countdown and minimum players, a fixed roster and spawning.
  - Alive, Eliminated, Spectate and Return to Hub, and whole-Kit drops.
  - A last-living-player win, the timeout, results and intermission, and the session log lines.
- **Why before the map:** it touches the Hub, the roster, spawning and the character lifecycle, so it's built and debugged in the current, well-understood arena before the new map exists. It's deliberately the largest and riskiest ticket, and isn't split in advance.
- **If it proves materially too large,** split it like this, and broaden neither half:
  - **17A, Match lifecycle:** the temporary Hub, countdown, fixed roster, spawning, Alive and Eliminated, whole-Kit drops, win and timeout.
  - **17B, Spectate and results:** Spectate, cycling between living players, Return to Hub, results and intermission.

### 18. Match map, Caches and Sentries

- **Scope:**
  - The roughly 250 × 250 greybox and its spawn points.
  - Breakable Rift Caches with crack stages.
  - Stationary Sentries with their Telegraph, kept thin on the real Tidal Bolt path, with no moving monster system.
  - The shuffled Ability bag, the early-variety adjustment, and central Caches' two drops set apart.
- **Starts after** the Match lifecycle works in the existing arena.

### 19. Rift Hook and Stone Guard

- **Scope:**
  - Rift Hook, with forced displacement toward its source and a pull that geometry stops.
  - Stone Guard, with server-authoritative Shield absorption.
  - Rank 2 values and specs for the sequence and burst limits.
  - Both added to the loot bag.
- **Starts only after** the loop works with the existing Reference Abilities, so new content adds to a working loop rather than hiding architectural problems.

### 20. Run test readiness

- **Scope:**
  - Stage 1 and 2 regression on the whole progression loop, with forged and flooded `ReplaceRequest`s and the other authority and security checks.
  - A Reference Kit regression.
  - Pacing logs: time to first Ability and to a full Kit, Upgrades, replacements, central encounters, eliminations and timeouts.
- **Publishing:** this prepares a publish candidate and stops for the developer's approval. Nothing is published as part of finishing the ticket.
- **The sitting with outside testers** still waits for Stage 3 to validate the combat.

## Later, after evidence from the run test

- non-Ability world items, under the principles above
- the smallest Rift Collapse, if timeouts are more than occasional
- monsters, Elites and Rank 3 behaviour changes
- the real Hub, with batches of players into several Matches at once
- testing two slots against three, once there are about 6–8 Abilities
