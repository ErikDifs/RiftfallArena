# Milestone 2 tickets

The ordered work for the second milestone: the **run progression loop**, **explore and fight PvE → find Abilities → change the Kit → fight players with it**. It tests whether Riftfall gets more interesting when a player's Kit develops during a run, not whether more content can be added.

Being settled from 2026-11-08, before any building. Work stops after each ticket until the developer gives the go-ahead, as in [Milestone 1](milestone-1-tickets.md).

## Checkpoints

| Date | Checkpoint | Contents |
|---|---|---|
| 2026-11-08 | Published place `134091394433436`, version 51 (private) | Ticket 15: the per-player Kit, its attributes and HUD, the Equip Lock, `ChargingRank` and the Starting Kit setting, Reference by default |
| 2026-11-09 | Published place `134091394433436`, version 70 | Tickets 16 and 17: Pickups and replacement, and the one-life Match with its Hub, Spectate, HUD and results, dormant behind the Reference Kit by default |
| 2026-11-09 | Published place `134091394433436`, version 79 | Ticket 18: the `MatchMap`, Rift Caches, Sentries and the shuffled Ability bag |
| 2026-11-09 | Published place `134091394433436`, version 86 | Ticket 19: Rift Hook and Stone Guard, the `Self` form, forced movement with its own duration, the Shield, and the four-Ability loot bag |
| 2026-11-10 | Published place `134091394433436`, version 98, from commit `8f027c8` | Ticket 20: the run-test telemetry and its specs (427), the Kill and Reaction confirm sound changes. Reference-safe: `startingKit = "Reference"`, `maxSeconds = 480`, no temporary aids |
| 2026-11-11 | Published place `134091394433436`, version 125, docs at commit `ce1ccff` | Ticket 21 (the Rift Crown, built and measured, human play checks postponed) and Ticket 22 steps 1 and 2 (`CollapseRules`, the `Collapse` runtime on a fixture, 574 specs). Reference-safe: `startingKit = "Reference"`, `maxSeconds = 480`, nothing requires the Collapse yet, no temporary aids |
| 2026-11-12 | Published place `134091394433436`, version 156, docs at commit `f411904` | Ticket 22 complete (the Rift Collapse, steps 1 to 10, 699 specs, S1 and S2 on the real map, settle 1.75 and the boundary rule). Published by the developer from the live editing session |

Roblox version history is the code's history, as [Milestone 1's checkpoints](milestone-1-tickets.md#checkpoints) set out, and only versions marked Published count.

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
- Ticket 19's two new Abilities get sensible Rank 2 values, and no Rank 3 behaviour.
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

- At the start of each Match, every player's Kit is empty and the world's Pickups are placed fresh. There are no Gather, Clash or Rift Collapse phases. (Updated 2026-11-09: on the Rift Crown, [Ticket 22](#22-rift-collapse) adds a time-driven Rift Collapse inside Running. The Match lifecycle is unchanged, and Gather, Clash and Collapse are derived labels, not states.)
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
- *(Superseded 2026-11-09: [Ticket 22](#22-rift-collapse) specifies the Rift Collapse for the Rift Crown, and the Timeout remains a failsafe. The text below is the original reasoning.)* No Rift Collapse in this milestone. Convergence comes from a small enough map, loot placement, contested valuable areas, the noise of fighting, and Eliminated players' whole-Kit drops worth contesting.
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

**The two new Abilities: Rift Hook and Stone Guard (2026-11-08).** Ticket 19 isn't just two more Abilities: it tests whether finding different pairs changes how players approach a run and a fight. Both use the existing Instant Casting Style. No new Charging Ability and no new Status chain. All numbers are Ticket 19's starting tuning.

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

**Settled 2026-11-08, before building.**

- **Replication.** The server system `Kits` owns each player's Kit and writes it to the `Player` as attributes: `KitSlot1Ability`, `KitSlot1Rank` and `KitSlot1LockedUntil` (server time), and the same for slot 2. They reach every client, late joiners included, with no new remote. `Charging` also sets `ChargingRank` beside `ChargingAbility` and `ChargingSince`, from the Kit when Charging starts, and other players' Telegraphs read it. Both are presentation only: the Cast's Rank is still resolved on the server at release.
- **Starting Kit.** `Tuning.kit` holds the slot keys `{ "Q", "E" }`, `startingKit = "Reference"` or `"Empty"`, the Reference Kit's Abilities, `maxRank = 2` and `equipLockSeconds`. `inputKey` and `slot` leave `Config.Abilities`, and `Tuning.prototypeKit` goes. The Starting Kit is applied on every spawn, at Rank 1 and with no Equip Lock, until Ticket 17 moves it to entering a Match. The Basic Attack is never a slot and is always available.
- **HUD.** The Reference HUD keeps the coloured squares and key letters, and adds each Ability's name and Rank pips. An empty slot is a dark outlined square with its key and "Empty", and pressing it sends nothing.
- **`KitRules` for this ticket.** Pure functions on a plain Kit table: `new`, `starting`, `find`, `firstFreeSlot`, `equip` (into an empty slot only, setting the Equip Lock, refusing a Rank over the cap), `canCast` (`NotInKit` or `EquipLocked`), and `toAttributes` and `fromAttributes`, which the client and server share. Fits, Upgrades, replacement, drops and `ReplaceRequest` wait for Ticket 16.
- **Equip Lock.** `Casts` asks the Kit before the cooldown on `CastRequest`, `ChargeStart` and `ChargeRelease`, and rejects a locked Cast as `EquipLocked` with the time left. `equipLockSeconds` starts at 0.85. On the HUD a locked slot dims, gets a white outline, and a thin bar fills across its bottom, so it reads differently from the cooldown's vertical shade. The key letter stays.
- **Granting before Pickups.** A server-only `Kits.grant(player, abilityId, rank)` puts an Ability into the first free slot with the Equip Lock, and Ticket 16 reuses it. Specs cover the logic through `KitRules`. Play tests call `grant` through a TEMPORARY BindableFunction, with `startingKit` set to Empty in Edit mode and reverted afterwards.
- **Rank cap.** `RankRules.MAX_RANK = 3` stays as the range the Ability data supports, so the Rank 3 values stay valid. `Tuning.kit.maxRank = 2` is the gameplay cap that `KitRules` enforces. How Upgrades read it is Ticket 16's.
- **Specs.**
  - **Changed:**
    - "Arc Burst is in the Kit, on E" becomes: the Reference Kit has Arc Burst in slot 2, and slot 2 is E.
    - "no two share a key or slot" becomes: the slot keys are distinct, and none is LeftShift or F.
    - The three `prototypeKit` loops cover every Ability in Config at every Rank up to the cap.
  - **New `KitRulesSpec`:**
    - the Reference and Empty Kits
    - `firstFreeSlot` gives Q, then E, then none
    - `equip` sets the lock, and refuses an occupied slot, an unknown Ability and a Rank over the cap
    - `canCast` returns `NotInKit`, then `EquipLocked`, then OK once the lock ends
    - the attributes round-trip
    - the cap is at most `MAX_RANK`

**Built 2026-11-08.**

- **New:** `KitRules` (shared Rules) and `Kits` (server), with `KitRulesSpec`.
- **`Tuning.kit`** replaces `Tuning.prototypeKit`, and `inputKey` and `slot` are gone from `Config.Abilities`.
- **`Casts`** asks `Kits.canCast` for every request. Its cooldown table is renamed `cooldownsByPlayer` and stays per Ability per life. A release whose Ability has left the Kit is logged as a refused release.
- **`Charging`** sets `ChargingRank`, and `Protocol` names it.
- **Client:**
  - `Input` reports slots.
  - `Casting` keeps a copy of the Kit from the attributes, refreshed on `AttributeChanged`, and sends nothing for an empty slot or during an Equip Lock.
  - `Hud` builds slots by number, with the name, Rank pips out of `maxRank`, the empty state and the Equip Lock look. The slot gap grew from 6 to 18 so the names fit.
  - `ChargeTelegraphs` takes the Rank from the caller, or from `ChargingRank` for other players.
- **The pips are 8 px,** with a fainter outline on Ranks not yet reached, because at 6 px filled and hollow were hard to tell apart.
- **Specs:** the three `prototypeKit` loops and the two key tests changed as settled. The 9 KitRules tests make 207 specs, all passing.
- **`code-structure.md`** gains a Kits section.

**Run 2026-11-08 by `studio-tester`, solo and then on 2 clients, reviewed by the main session. All 27 checks pass, and no code changed during testing.** Real input ran at about 15 fps. The Empty checks set `startingKit = "Empty"` in Edit mode and reverted it after, which was confirmed. They granted Abilities through a TEMPORARY `Kits.grant` hook, deleted afterwards, and a search for TEMPORARY now finds nothing.

**Reference Kit, solo (against version 46):**

| # | Check | Result |
|---|---|---|
| A1 | Specs | 207 passed, 0 failed |
| A2 | Kit attributes and HUD | Tidal Bolt/Q and Arc Burst/E at Rank 1, with one filled pip and one hollow, no Equip Lock, and no `LockedUntil` attributes |
| A3 | Held Basic Attack | 6 hits of 6, 0.40–0.42 s apart, with no Knockback |
| A4 | Tap Q, then tap E | 12.034, then 30.02 (Wet Reaction) |
| A5 | Q held 0.65 s, then 2.6 s | Walking 16 → 12, with `ChargingRank` 1 next to `ChargingAbility`. All cleared on release, which did 13.45. The 2.6 s hold auto-released for 15 |
| A6 | Tap Q, then E held 2.2 s | 12, then 32.1 |
| A7 | Cooldown | A second Q tap 0.8 s later did nothing. The key counted down and the shade drained |
| A8 | Death and respawn | The Kit and HUD were unchanged, and Q worked straight away |
| A9 | Forged requests | A `CastRequest` with an extra `rank` was dropped as malformed. An unknown Ability gave `NotInKit`, Tidal Bolt through `CastRequest` gave `NotInstant`, and a Basic Attack `ChargeStart` gave `NotChargeable` |

**Empty Kit with grants, solo:**

| # | Check | Result |
|---|---|---|
| B1 | Empty Kit | No Kit attributes. Both slots read "Empty", dark, with Q and E and no pips |
| B2 | Pressing empty slots | Taps and 1 s holds sent nothing and drew nothing. The Basic Attack still hit for 6 |
| B3 | Grant Tidal Bolt, Rank 1 | Q slot. The Equip Lock showed for 0.83 s at 15 fps: dimmed to 0.55 with a 2 px white outline and the bar filling, then back to normal |
| B4 | Grants that must fail | Rank 3 gave `InvalidRank`, the Basic Attack gave `UnknownAbility`, and a duplicate gave `AlreadyOwned` |
| B5 | Q after the lock | 12 |
| B6 | Grant Arc Burst, Rank 2 | E slot with both pips filled. Q then E did 12, then 33.6 (Rank 2's 16 plus a Reaction bonus of 16 × 1.1). An E hold showed `ChargingRank` 2 and did 17.87 |
| B7 | Grant into a full Kit | `KitFull` |
| B8 | Respawn with an Empty Starting Kit | The Kit was empty again |
| B9 | Server Equip Lock | A `ChargeStart` sent the moment the grant arrived was rejected as `EquipLocked`, with 0.784 s left, and no Charging started |

**Reference Kit, Player1 and Player2:**

| # | Check | Result |
|---|---|---|
| C1 | Player1's full Q, seen by Player2 | `ChargingAbility` TidalBolt and `ChargingRank` 1 appeared, then cleared. The orb grew to 4.2. `CastStarted` had rank 1 and charge 1, and the hit was 15 |
| C2 | Player1's full E, seen by Player2 | `ChargingRank` 1 and the lightning light appeared, with no ground preview on Player2. The hit was 16.1 |
| C3 | PvP | Tap Q then full E did 12, then 32.1. Basic Attack hits were 6, 0.40 s apart |
| C4 | Both HUDs and attributes | Each showed its own Reference Kit, and the other player's Kit attributes replicated |
| C5 | Kill and respawn | `killed=true`. Player2 respawned 3.02 s later with the same Kit and Charged Q straight away, and Player1 saw `ChargingRank` 1 |
| C6 | Forged `ChargeStart` with `rank = 2` | Dropped as malformed, with no Charging and no `ChargingRank` |

**Accepted with notes:**

- **The client predicts the Equip Lock from server time.** A slow clock could send one request a moment early. The server refuses it as `EquipLocked`, and the client then shows the time left as a short cooldown shade. It's rare, and presentation only.
- **The Telegraph's Rank comes from when Charging starts,** while the Cast's Rank is resolved at release. An Upgrade during a Charge, possible from Ticket 16, would draw the old Rank until release. That's presentation only and accepted.
- **Test setup, not bugs:** the camera's yaw and pitch drifted in the solo run and were pinned with a TEMPORARY `BindToRenderStep`. A server-side `PivotTo` was reverted by Movement Validation. A first Q then E from about 30 studs was pushed out of Arc Burst's range. All three are in the quirks doc.

**Published by the developer on 2026-11-08 as version 51**, recorded in the checkpoints above.

### 16. Pickups and replacement

- **Scope:** server-owned Ability Pickups, collection by proximity, Upgrades, F+Q and F+E replacement through `ReplaceRequest`, the replaced Ability dropping, and readability in the world and HUD.
- **Where the logic lives:** fit, Upgrade and replacement decisions go in `KitRules` where practical. World proximity, instance ownership and replication stay out of the pure Rules.
- **Tested with** Pickups placed by hand in the current arena and an Empty Starting Kit.

**Settled 2026-11-08, before building.**

- **A Pickup in the world.**
  - The server keeps each Pickup's record: id, Ability, Rank and anchor position.
  - The world gets an anchored, invisible Part in `Workspace.Pickups`, with no collision, touch or query. It's tagged `Pickup` and carries `PickupId`, `PickupAbility`, `PickupRank` and `PickupSpawnedAt` (server time). Ids come from a counter that only goes up and are never reused.
  - Clients draw everything else in a new `PickupVisuals`, and it's presentation only: it never affects collection, line of sight or collision.
    - A shape for each Ability from `visual.pickupShape`: a sphere for Tidal Bolt (about 1.6 studs) and a cube on its corner as a diamond for Arc Burst (about 1.3). Cube is supported too, for later Abilities. Shapes are Neon in the Ability's colour, bob ±0.3 studs about every 2 s, turn 90° a second from one shared loop, and have a small light with a range of about 8.
    - A BillboardGui 2.2 studs up, with the name in GothamBold at 18 px and Rank pips out of `maxRank` in the HUD's style. It's visible to 40 studs and not `AlwaysOnTop`.
    - At Rank 2, a flat ring at the base in the Ability's colour brightened toward white, with an outer radius of about 1.4 and an inner one of about 1.2. If `CylinderHandleAdornment` doesn't give a clean ring, the simplest client-only equivalent will do.
    - A beam 30 studs tall, 0.25 wide and 0.7 transparent, with no shadow or interaction, hidden beyond 150 studs.
    - A 3-stud burst with no sound when a Pickup appears within `appearFlashSeconds` (1) of `PickupSpawnedAt`, so Pickups already there when a player joins don't flash.
  - All of these values are starting tuning, in `Tuning.pickups.display`.
- **KitRules' decisions.** `RankRules.MAX_RANK` is what the Ability data supports, `Tuning.kit.maxRank` is what the game mode permits, and only KitRules enforces the gameplay cap.
  - `fit(kit, abilityId, rank, kitConfig)` gives `Equip` with the first free slot, `Upgrade` with the slot and new Rank, `AtCap`, or `NeedsReplace`.
  - `upgrade` sets the Rank to `min(maxRank, RankRules.upgradedRank(...))`. It never sets an Equip Lock and leaves a running one as it is.
  - `replace` works only for `NeedsReplace`. It puts in the new Ability with a fresh Equip Lock and returns the old one at its Rank. It refuses with `NoSuchSlot`, `SlotEmpty`, `UnknownAbility`, `InvalidRank`, `Fits` or `AtCap`.
  - A Pickup above the cap can't be made.
- **Collection.** A server `Pickups` system checks every `checkSeconds` (0.1), never with `Touched`.
  - It measures 3D distance from the server's `HumanoidRootPart.Position` to the anchor, which sits about 3 studs above the floor. `collectRadius` is 4.
  - Collectors must be alive and not `Stunned` or `Frozen`. Charging and Rooted players can collect.
  - Pickups are handled in ascending id order. For each, `PickupRules.chooseCollector` picks the nearest eligible player for whom it fits as `Equip` or `Upgrade`, with exact ties going to the first in iteration order.
  - `Kits.collect` reruns `fit` against the current Kit and applies only `Equip` or `Upgrade`. The Pickup is removed in the same step, and the Kit is up to date before the next Pickup is checked.
  - `AtCap` and `NeedsReplace` Pickups stay in the world.
  - An Upgrade during a Charge changes the Kit at once, and the release resolves the new Rank. The Charge isn't restarted or changed, and its Telegraph may keep showing the Rank it started with.
  - Session counts `Equip`, `Upgrade` and `Replace`.
- **ReplaceRequest `{ pickupId, slot }`.** The server reads the Ability and Rank only from its own record, so a request that adds `rank` or `abilityId` is malformed.
  - **The guard.** Before anything else, a per-player token bucket (`replaceGuard`: a burst of 5, refilling 5 a second) drops excess requests. Nothing in the game had flood protection to reuse, and normal play can't hit this limit.
  - **The checks,** in this order, against the server's state when the request is handled:
    1. the payload's shape
    2. `NoCharacter`
    3. `Stunned`
    4. `Frozen`
    5. `Charging`, which leaves the Charge running
    6. `Gone`
    7. `TooFar`, beyond `replaceRadius` (6) in 3D
    8. `Kits.replace`
  - `PickupRules.validateReplace` holds checks 2–7 as pure logic.
  - **On success,** in one synchronous step: the Pickup is removed, the slot is replaced, the old Ability drops as a new Pickup, the Kit is published, Session counts a `Replace`, and `PickupCollected` fires. The first valid request wins, and later ones see `Gone`.
  - **What doesn't block it:** a Dash, a cooldown on either Ability, or an Equip Lock on the other slot. Cooldowns stay per Ability per life.
  - **Refusals** change nothing and get no reply. The warning for refusals and dropped requests is throttled to about one a second per player.
  - **The drop** lands from a raycast straight down from the server's root position, against `Workspace.Arena` only, up to 20 studs, with the anchor 3 studs above the hit, or at the root position with no hit. It gets a new id and keeps its Rank. The replacer, whose Kit is full, can't collect it automatically, but can swap back with F, which costs a fresh Equip Lock. Another player it fits can collect it on a later check.
- **`PickupCollected { pickupId, abilityId, rank, collectorId, kind, slot, position }`,** sent to all clients.
  - `kind` is `Equip`, `Upgrade` or `Replace`, and `rank` is the Rank afterwards, presentation only.
  - The server publishes the Kit attributes, fires the event and destroys the Part, in that order. Clients remove a Pickup's visuals on the event or the tag's removal, whichever comes first, and doing it twice is harmless.
  - The event is only for feedback. The client's Kit comes only from the attributes.
  - There's no refusal event and no spawn event.
- **The client's F modifier.** `Tuning.kit.replaceModifier = "F"`.
  - `Input` counts F as held from a press the game hasn't already handled, until F comes up or the window loses focus.
  - While F is held, Q and E presses go to `onReplacePressed(slot)` and never to Casting. Releases always reach `onSlotReleased`, so a Charge started before F releases normally.
  - Letting go of F while Q is held doesn't start Charging, so a fresh press is needed. F alone affects nothing.
  - A new client module, `Replacing`, keeps its own small copy of the Kit from the attributes, rather than sharing code with `Casting`.
    - Each frame it targets the nearest `NeedsReplace` Pickup within `replaceRadius` with `PickupRules.chooseReplaceTarget`, with ties going to the lower id.
    - It has no target while the player is dead, Stunned, Frozen or Charging, using a new read-only `Casting.isCharging()`.
    - With a target, F with Q or E sends one `{ pickupId, slot }` and predicts nothing. Without one, it sends nothing.
  - **The prompt.** Two lines above the slots, such as `F + Q — Replace Tidal Bolt with Arc Burst`, faded at about 0.4, and fully opaque while F is held. The target gets a local white outline that walls hide.
- **Readability.**
  - Everyone sees a 4-stud burst in the Ability's colour at `position`, and hears a new `PickupCollect` sound from there, up to about 90 studs. The collector hears it only flat.
  - The slot pulses 1 → 1.15 → 1 over 0.25 s for Equip, Upgrade and Replace.
  - An Upgrade shows `Upgrade → Rank 2` above the Crosshair in the Ability's colour at 24 px, or `Upgrade: Rank 2` if the font can't draw the arrow.
  - Equip and Replace get no text.
  - The sound is a built-in placeholder that isn't already used and reports `IsLoaded`. No asset is uploaded.
  - Out of scope: an appear sound, marking a Pickup that's at the cap, rarity effects, minimap markers and icons.
- **Placed for testing.**
  - Marker Parts in `ServerStorage.PickupPlacements`, with `Ability` and `Rank` attributes, become Pickups at server start, but only when `startingKit == "Empty"`.
  - Collected Pickups don't return until Play restarts. Ticket 17 moves placement to the start of each Match.
  - The six markers, with anchors 3 studs above the floor:
    1. Tidal Bolt, Rank 1, at (30, 3, -20)
    2. Arc Burst, Rank 1, at (20, 3, -40)
    3. Tidal Bolt, Rank 1, at (0, 3, -40)
    4. Arc Burst, Rank 2, at (-12, 3, -40)
    5. Tidal Bolt, Rank 2, at (-28, 3, 42)
    6. Arc Burst, Rank 1, at (30, -5, 30)
  - **Reaching a replacement in Play.** With only two Abilities, a full Kit can't meet a `NeedsReplace` Pickup. So for the Play checks only, `studio-tester` adds a TEMPORARY `TestShard` to `Config.Abilities`: Tidal Bolt's stats, a non-blue colour and `pickupShape = "Cube"`. It also adds TEMPORARY markers for it at Rank 1 at (20, -5, 40) and Rank 2 at (25, -5, 40). Everything is reverted afterwards and confirmed with a search. It's never committed. The TEMPORARY `Kits.grant` hook is still allowed.
- **Specs.**
  - **Changed:** the slot-key test reads `replaceModifier`.
  - **New in KitRulesSpec:**
    - every `fit` result
    - the cap from config, at 2 and 3
    - an Upgrade adds no Equip Lock and keeps a running one
    - `replace`, with every refusal, using a fake third Ability
  - **New PickupRulesSpec:**
    - `chooseCollector`, including the ledge case and the next nearest
    - `validateReplace`, every reason plus Stunned and TooFar together giving `Stunned`
    - `chooseReplaceTarget`
    - `takeToken`
    - `dropPosition`
  - **New PickupsSpec,** with fake dependencies:
    - Pickups handled in ascending id order
    - two collections in one check
    - collecting and removing in one step
    - a refused `Kits.collect` leaves the Pickup
    - a Pickup over the cap can't be made
  - **New ProtocolSpec cases:** a valid `ReplaceRequest`, an added `rank` or `abilityId`, and non-integer, NaN or infinite values; a complete `PickupCollected`.
  - **New Config specs:** every slot Ability has a supported, unique `pickupShape`; `replaceRadius` is greater than `collectRadius`.
- **Play checks,** run by `studio-tester`:
  - **A, Reference Kit, solo:** Ticket 15's regression, plus no Pickups and a forged `ReplaceRequest` refused.
  - **B, Empty, solo, with the markers:** visuals, Equip into Q and E, Upgrade during a Charge, at the cap, Stunned, the arrow glyph and the sound.
  - **C, Empty, solo, with TestShard:** the prompt and target, F+Q and F+E, the drop, swapping back, cooldowns, the F input behaviours, forged refusals, and the 100-request flood (5 handled, 95 dropped by the guard, one replacement and one drop).
  - **D, Player1 and Player2:** first a short Reference Kit check (both Kits, one Tidal Bolt into Arc Burst exchange, the remote Telegraph's Rank, no Pickup visuals), then Empty with TestShard: contested collection, the dropped Ability collected by the other player, the same Pickup requested at once, and the prompt and outline seen only locally.

**Built and tested 2026-11-08. All checks pass.**

- **Server side:**
  - `KitRules` gained `fit`, `upgrade`, `replace` and `isValidRank`.
  - The new `PickupRules` is pure logic: `chooseCollector`, `validateReplace`, `chooseReplaceTarget`, the token bucket and `dropPosition`.
  - The new `Pickups` server system handles records, spawning, the 10 Hz collection check, `ReplaceRequest`, the throttled warnings and placements.
  - `Kits` gained `get`, `collect` and `replace`. `Session` counts and logs Pickups.
  - `Protocol` gained `ReplaceRequest`, `PickupCollected` and the Pickup attribute names. The Tuning values are in `kit.replaceModifier` and `pickups`.
- **Client side:**
  - `Input` gained the F modifier, and `Casting` gained `isCharging`.
  - The new `Replacing` module drives the prompt, the target and the request.
  - `studio-implementer` built the new `PickupVisuals`, the Hud prompt and slot pulse, the Feedback burst, sound and Upgrade text, the `pickupShape` values and the specs. All of it was reviewed before acceptance.
- **Placements:** six markers in `ServerStorage.PickupPlacements`.
- **Specs:** 249 passed, 0 failed, run on the Server in Play.
- **Test setup:** a TEMPORARY TestShard, two shard markers and a TEMPORARY hook (grant, spawn, Kit read). All were removed after the runs, a search for `TestShard` and `TEMPORARY` finds nothing, and `startingKit` is back to `"Reference"`. After the clean-up, specs passed 249 of 249 again, with the Reference Kit and no Pickups.

**A, Reference Kit, solo:**

| # | Check | Result |
|---|---|---|
| A1 | Specs | 249 passed, 0 failed |
| A2 | Kit and prompt | Tidal Bolt/1 and Arc Burst/1, no `LockedUntil`, `ReplacePrompt` hidden |
| A3 | No Pickups | `Workspace.Pickups` and the client's `PickupVisuals` were both empty |
| A4 | Tap Q, then E | 12.037, then 30.021 (Wet Reaction) |
| A5 | Q held 0.65 s | Walking 16 → 12, `ChargingRank` 1, cleared on release, 13.45 |
| A6 | Cooldown | A second Q tap 0.8 s later gave no second hit |
| A7 | Death and respawn | Kit unchanged, and a charged Q hit for 13.45 straight away |
| A8 | Forged requests | A `CastRequest` with `rank` was dropped as malformed. `ReplaceRequest {1, 1}` was refused as `Gone`. Kit and Pickups unchanged |
| A9 | F held with other inputs | The Basic Attack still hit, 4 hits each for 6, and LeftShift still Dashed about 21 studs |

**B, Empty Kit, solo, with the markers:**

| # | Check | Result |
|---|---|---|
| B1 | Visuals | Six Pickups at the right places, Abilities and Ranks. Tidal Bolt: a Neon sphere of 1.6, label, pips, beam 0.25 × 30 at 0.7. Arc Burst Rank 2: a diamond of 1.3, both pips filled, and the base ring (1.4/1.2). In a capture the ring reads clearly as a ring, so the fallback wasn't needed |
| B2 | Equip into Q | Tidal Bolt into Q with a 0.83 s Equip Lock shown on the slot, which pulsed to 1.15. No text. The Pickup and its visual were gone, and Session logged `Equip` |
| B3 | Equip into E | Arc Burst/1 into E with a lock and pulse |
| B4 | Upgrade Q | Rank 1 → 2, `LockedUntil` unchanged, "Upgrade → Rank 2" at 24 px in blue. Session logged `Upgrade` |
| B5 | Upgrade E | Arc Burst Rank 2 with the pulse and yellow text |
| B6 | At the cap | Standing on Marker5 at Rank 2 for 2.2 s changed nothing |
| B7 | Upgrade during a Charge | Collected 0.4 s into a Q hold. `ChargingRank` stayed 1, and the release hit for 15.91, Rank 2's 13 × 1.224. Rank 1 would be 14.69 |
| B8 | Stunned | The Pickup stayed for 1 s, then was collected 0.03 s after the Stun cleared |
| B9 | Sound | `PickupCollect` is `rbxasset://sounds/snap.mp3`, at speed 1.8. It reports `IsLoaded` with a length of 0.418 s |
| B10 | Full Kit with F at a capped Pickup | No prompt, no outline |

**C, Empty Kit, solo, with TestShard:**

| # | Check | Result |
|---|---|---|
| C1 | Prompt and target | "F + Q — Replace Tidal Bolt with Test Shard" and the E line, at 0.4 transparency, opaque while F was held. The outline moved to the nearer Pickup, and both were gone at 8.5 studs |
| C2 | F+Q | Q became Test Shard with a 0.85 s lock. The server got one `{pickupId, slot}`. Tidal Bolt R1 dropped at (21, -5, 40), 3 above the floor, and wasn't collected over 2 s. Session logged `Replace dropped=TidalBolt`. No Charging started |
| C3 | Swap back | Q became Tidal Bolt with a fresh lock, and a Test Shard dropped |
| C4 | F+E | E became Test Shard, and Arc Burst R1 dropped |
| C5 | Cooldowns per Ability per life | After a tap, then swapping out and back in, a `ChargeStart` once the lock ended was refused as `OnCooldown` with 1.55 s left |
| C6 | F input behaviours | F then Q held: no Charge. Q held, then F, then Q released: Cast normally. Releasing F with Q still held: no Charge. A fresh Q press then Charged. F+Q and F+E with no target sent nothing |
| C7 | Forged refusals | Extra `rank` or `abilityId`: malformed. Unknown id: `Gone`. 7 studs: `TooFar`. While Charging: `Charging`, and the Charge kept going. Slot 3: `NoSuchSlot`. Own Rank 1 Ability: `Fits`. Own Rank 2: `AtCap`. Nothing changed each time |
| C8 | 100-request flood | One replacement, one drop and one Session line. One warning, then "(98 more held back)": 3 `Gone` and 95 dropped by the guard |
| C9 | Stunned | Refused as `Stunned`, Kit unchanged |

**D, Player1 and Player2:**

| # | Check | Result |
|---|---|---|
| D0–D6 | Reference Kit | 249 specs passed. Both Kits replicated, with no Pickups or prompt. Tidal Bolt then Arc Burst did 12, then 30 with Wet, on both clients. Player2 saw Player1's `ChargingRank` 1, and `CastStarted` had rank 1. E's hold showed `ChargingRank` 1. No errors |
| D7 | Empty start | Both Kits empty. 8 Pickups, and 8 visuals on each client |
| D8 | Contested collection at Marker6 | Both landed in the same server tick, at 2.0 and 3.5 studs. The nearer Player1 got Arc Burst with a lock and the slot pulse. Player2's Kit stayed empty, and Player2 heard the sound at (30, -5, 30). One Session `Equip` line, and the visual was gone on both clients |
| D9 | The drop goes to the other player | With real input, Player1 pressed F+E. E became Test Shard, and Tidal Bolt R1 dropped under Player1, visible on both clients. Player2 walked onto it and equipped it |
| D10 | Same Pickup at once | Both sent `{pickupId = 11, slot = 1}` 5 ms apart. Exactly one replacement (Player2's) and one drop. Player1 was unchanged and refused as `Gone`. One Session `Replace` line |
| D11 | Prompt and outline local only | Player1 saw both. Player2, far away or 5 studs away with a Kit that wasn't full, saw neither |
| D12 | Consoles | No errors on the server or either client |

**Accepted with notes:**

- **Starting values.** The look, the arrow glyph and the sound are starting values. The ring and "Upgrade → Rank 2" both rendered cleanly, so neither fallback was used. The `PickupCollect` sound is the built-in `snap.mp3` at speed 1.8, a placeholder, and no asset was uploaded.
- **Telegraph during an Upgrade.** An Upgrade during a Charge resolves the new Rank on release (B7), while the Telegraph keeps the Rank it started with, as settled.
- **Session log names.** Session lines name players by UserId (`player:-1`), not by name.
- **Test setup, not bugs:**
  - The developer's own input in Studio stopped Play twice and added Casts during the first solo run.
  - The ShootingDummy hit the tester on the way to markers.
  - A test hop collected the Rank 2 TestShard on its way past, so a few C7 refusals used an Arc Burst Pickup as the target.
  - Specs called from Edit mode show false failures from stale module copies.
  - All of these are in the quirks doc.

**Published by the developer on 2026-11-09 as version 70**, recorded in the checkpoints above.

### 17. One-life Match

- **Scope:**
  - The temporary Hub, the countdown and minimum players, a fixed roster and spawning.
  - Alive, Eliminated, Spectate and Return to Hub, and whole-Kit drops.
  - A last-living-player win, the timeout, results and intermission, and the session log lines.
- **Why before the map:** it touches the Hub, the roster, spawning and the character lifecycle, so it's built and debugged in the current, well-understood arena before the new map exists. It's deliberately the largest and riskiest ticket, and isn't split in advance.
- **If it proves materially too large,** split it like this, and broaden neither half:
  - **17A, Match lifecycle:** the temporary Hub, countdown, fixed roster, spawning, Alive and Eliminated, whole-Kit drops, win and timeout.
  - **17B, Spectate and results:** Spectate, cycling between living players, Return to Hub, results and intermission.

**Settled 2026-11-08, before building.**

- **Switch and split.**
  - The Match runs only when `Tuning.kit.startingKit == "Empty"`. There's no second flag and no mode framework.
  - With the Reference Kit, Ticket 17 stays dormant: no Hub, Match, Spectate or results. `Characters` respawns on the arena pads after about 3 s, the Reference Kit is granted, and the dummies stay.
  - It stays one ticket, built in two stages:
    - **17A:** the Hub, phases, roster, spawning, elimination, whole-Kit drops, the end grace, the endings and clean-up, with its specs and a solo Play check.
    - **17B:** Spectate, Return to Hub, and the HUD, feed, banner and results.
  - If 17A proves materially larger or riskier, it's recorded as its own ticket, and 17B is settled again before it's built. Neither half gets broader.
  - **Pickups belong to the Match.** There are none while Waiting. At Match start, `Pickups.clearAll()` runs, then `Pickups.placeMarkers()` places Ticket 16's six markers fresh. At Match end every Pickup is cleared, including replacement and elimination drops.
  - **Not in this ticket:** the final Hub, `TeleportService`, batching Matches, Caches, Sentries, shuffled loot, the Rift Collapse and new Abilities.
- **Who owns what.**
  - **The new pure `MatchRules`** has `step`, `buildRoster`, `assignSpawns`, `eliminate`, `checkEnd`, `stateOf`, `canAct` and `minPlayers`. It uses no Roblox objects.
    - `checkEnd` is the only place that decides an ending, and `step` uses its result.
  - **The new `Match` server system** is the only owner of participation. It runs a 0.25 s loop for phases. Eliminations happen when they arrive, not on the loop.
    - With the Reference Kit, `Match.start` returns at once and publishes nothing.
  - **`Characters` keeps the mechanics, and `Match` sets the policy.**
    - New functions: `setSpawnPolicy(fn)` (Hub, Arena or None), `spawnInArea(player, area, pad?)`, `removeCharacter(player)`, `onDeath(callback)` and `lastPosition(player)`.
    - `onDeath` reports `Died` or `Removed` (destroyed without dying), with the last root position.
    - Administrative removals and reloads go through the existing expected-removal mechanism and never call back. **Replacing a character as part of the Match lifecycle is not a death.**
    - Without a policy, `Characters` behaves exactly as it does today.
  - **`Kits`** gains three things:
    - `takeAll(player)`: a transfer. It empties the Kit, publishes it, and returns `{ abilityId, rank }` in slot order.
    - `resetForMatch(player)`: an explicit empty Kit, owned by Match entry rather than left to a `CharacterAdded` side effect.
    - `onPlayerLeaving(fn)`: a handler `Kits` calls in its own `PlayerRemoving`, before deleting the record. `Match` runs the `Left` elimination inside it, so the Kit is always transferred before it's destroyed. `Characters` doesn't report `Left`.
  - **`Pickups`** gains `clearAll()` (no feedback), `placeMarkers()`, a `fromElimination` option that sets `PickupFromElimination`, and a `canCollect(player)` dependency.
  - **Replicated state** is for presentation only. The server always asks `Match`.
    - Workspace: `MatchPhase`, `MatchPhaseEndsAt` (server time), `MatchId`, `MatchAliveCount`.
    - Player: `MatchState` (`Hub`, `Alive` or `Eliminated`).
    - None of these are set with the Reference Kit.
- **The temporary Hub.**
  - `Workspace.Hub` is centred at (0, 0, 300). It's a 60 × 60 neutral greybox floor at y = 0 with 8-stud walls and the pads `HubSpawn1`–`HubSpawn4`. There's no route between it and the arena.
  - It has no Pickups, dummies, signs, menus, queue UI or `TeleportService`.
  - It sits unused with the Reference Kit, and its presence changes nothing there.
  - **Spawning there.** `spawnInArea(player, "Hub")` picks a pad with `SpawnRules.choose`. It's used when a player joins, after Return to Hub, after the results, and on a death in the Hub, after the usual 3 s.
  - A Hub character has full health and an empty Kit. A fresh character clears per-life state such as Charging and Statuses.
  - The walls are only for containment. `MatchState` and the server gates are what keep Hub players out of the Match.
  - This Hub is a test environment, not the target architecture.
- **Phases and roster.**
  - `Tuning.match` holds `minPlayers = 2`, `minPlayersInStudio = 1` (chosen with `RunService:IsStudio()`, and used for nothing else), `countdownSeconds = 15`, `maxSeconds = 480`, `resultsSeconds = 10`, `loopSeconds = 0.25`, `bodySeconds = 3` and `endResolutionSeconds = 0.25`.
  - **Waiting.** Eligible players are those with `MatchState = Hub` and a living character. Countdown starts at the minimum.
  - **Countdown** cancels below the minimum and restarts in full later. Anyone who arrives during it is in.
  - **Match start** happens in one step:
    1. Recount the eligible players. If there are fewer than the minimum, go back to Waiting.
    2. Fix the roster.
    3. `MatchId` + 1.
    4. `clearAll`, then `placeMarkers`.
    5. Deal the shuffled arena pads, one each. A roster bigger than four wraps around, as a limit of the current arena only.
    6. For each player: Alive, `resetForMatch`, then a fresh character on their pad.
    7. Running, with `MatchPhaseEndsAt = now + maxSeconds`.
  - There's no spawn protection. Nobody joins late, and Hub players and returners wait for the next Match.
  - **Results** lasts `resultsSeconds`, with no acting, collecting or being hit. Survivors can walk. At its end, in this order:
    1. Participants are no longer in the Match.
    2. `clearAll`.
    3. Fresh Hub characters.
    4. Waiting.
  - There's no separate intermission phase. All deadlines use `workspace:GetServerTimeNow()`.
- **Elimination.**
  - `Match` acts only for an Alive player in a Running Match.
    - `Died` covers combat, falls and resetting. A reset gives up the run.
    - `Removed` covers falling below `FallenPartsDestroyHeight`.
    - `Left` arrives through the `Kits` leave handler.
    - A death in the Hub just respawns there. A death during Results eliminates no one. A repeat elimination is a no-op.
  - **The killer** comes from a new `Session.killerOf(character)`, using the existing kill report. It's never invented for falls, resets or other deaths.
  - **The steps,** synchronous:
    1. `MatchRules.eliminate`.
    2. Publish the state and alive count.
    3. `takeAll`.
    4. Spawn the drops.
    5. The Session record.
    6. `PlayerEliminated`.
    7. `checkEnd`.
  - **The drop.** Each Ability keeps its Rank, whatever its Equip Lock. Two Abilities sit 1.5 studs either side of the origin, each placed by `dropPosition`.
    - The origin is the last root position. Without a floor under it, it's the last grounded position `Match` records every tick with a short downward raycast. If there's none, it's the safest fallback, logged.
    - Drops last until the Match ends.
  - **The body** stays `bodySeconds` (3), then is removed administratively. There's no respawn into the Match.
  - Kill Recovery stays at 25%, for valid player kills only.
- **Who can act,** in Match mode only. The server asks `Match` directly.
  - **`canAct`:** Running, on the roster and Alive.
    - `Casts` checks it for every `CastRequest` and `ChargeStart`, rejecting with `NotInMatch`. That covers the Basic Attack.
    - `ChargeRelease` checks again.
    - Leaving Running cancels every Charge with a new authoritative cause, `MatchEnded`.
    - `ReplaceRequest` refuses `NotInMatch` before its other checks.
  - **`canCollect`** has the same condition and goes to `Pickups`.
  - **`canBeHit`** has the same condition and filters `Characters.getTargets`, so Hub players, bodies, Results and anyone in the wrong place can't be hit. Damage, Statuses, Reactions and Knockback all follow from it.
  - **Attacks already in flight** from a player who has since been eliminated still resolve against targets that are valid at impact.
  - **Dash** stays usable everywhere. It's movement only.
  - **`Dummies.start`** returns early with an Empty Starting Kit.
  - **On the client,** when Match attributes exist and the local player isn't Running and Alive, `Input` passes no slot, Basic Attack or replacement presses on, and the slots dim. That's presentation only.
- **Ending.**
  - **`Winner`:** exactly one Alive player left, from a roster of at least 2.
  - **`NoSurvivors`:** none Alive.
  - **`Timeout`:** `now >= endsAt` with 2 or more Alive. There's no winner and survivors are listed, never picked by a secondary score.
  - **A solo roster** never wins. It ends as `NoSurvivors` or `Timeout`.
  - **The end grace.** When a roster of 2 or more first reaches one Alive player, `pendingEndAt = now + endResolutionSeconds`. This is internal, not a phase.
    - Nobody can act or collect during it, but Alive players can still be hit, so committed attacks land.
    - When it ends, one Alive gives `Winner` and none gives `NoSurvivors`.
    - `Match` schedules the resolution for that moment.
  - **Precedence.** Zero Alive with no pending end ends at once. A pending end, and any elimination, take precedence over Timeout.
  - **Running → Results:**
    1. Set the phase and its deadline.
    2. Cancel Charges with `MatchEnded`.
    3. Publish the attributes.
    4. `Session` logs the end.
    5. `MatchEnded`.
- **Spectate and Return to Hub,** 17B.
  - **The new client `Spectating`** is presentation only. It's active for `MatchState = Eliminated`, or for a Hub player who chooses **Spectate Match** during a Running Match.
  - **The camera.** `Camera.setSuspended(bool)`. Spectating uses the Custom camera on the target's Humanoid.
  - **Targets.**
    - During `bodySeconds` the camera stays on the player's own body.
    - Then it moves to the killer if Alive, otherwise to the first Alive player by UserId. When the target goes, it moves to the next.
    - With nobody Alive it holds the last view.
    - Q and E switch to the previous and next Alive player, by UserId and wrapping. There are also ◀ and ▶ buttons. `Spectating` owns those presses while it's active, and F has no effect.
  - **The Eliminated player's UI.**
    - The banner reads `ELIMINATED`, or `Eliminated by <name>`.
    - Below it, `Spectating <name> · <n> alive`.
    - A **Return to Hub** button.
    - No target Kit or cooldowns are shown.
  - **A Hub spectator** sees `Spectating <name>` and **Back**. Their character stays in the Hub. They leave Spectating by themselves when `MatchPhase` leaves Running, `MatchId` changes, or no target is left. **No spectator camera stays attached to a Match that has ended.**
  - **`ReturnToHub {}`** (ToServer) is accepted only from an Eliminated player in Running or Results.
    - It sets `Hub` and spawns them in the Hub administratively.
    - The elimination and the roster history stay as they were.
    - Everything else is refused silently, with a throttled warning and no token bucket.
- **Results and logs.**
  - **Stats.** `Session.beginMatch(matchId, roster)` snapshots each roster player's counters. The new pure `SessionRules.matchStats(snapshot, record)` gives the differences in kills, damage dealt, Reactions triggered, Equip, Upgrade and Replace.
  - **When rows are final.** At the elimination, including `Left`, or at the end for survivors. They're stored on the roster entry as history that nothing later changes.
  - **Rows** have the name captured at the start, and an outcome of `Winner`, `Survived` (Timeout) or `Eliminated`.
    - The place: the winner is 1st, and the Eliminated go in reverse order of elimination. Timeout survivors get "—".
    - The counts, and the time survived from the start of Running.
    - `Eliminated by <name>` only when there was a valid killer. `Left` may show as such. A `Died` with no killer shows just `Eliminated`, never a guessed "Fell" or "Reset".
  - **Log lines,** in the existing `[Riftfall][Session]` style and in addition to the existing ones:
    - `MatchStart` (match, players, roster)
    - `Eliminated` (match, player, cause, by, alive, seconds, and the dropped Abilities with Ranks in slot order)
    - `MatchEnd` (match, cause, winner, alive, seconds, eliminations): the key metric for timeouts
    - one `MatchResult` per roster player
    - `ReturnToHub`
- **Protocol and HUD,** 17B except for the remotes.
  - **Remotes:**
    - `ReturnToHub {}`, where any field is malformed.
    - `PlayerEliminated { matchId, playerId, killerId, cause, aliveCount, position }`, with an empty-string `killerId` for none.
    - `MatchEnded { matchId, cause, winnerId, durationSeconds, rows }`, sent once to everyone. If `Protocol` can't validate nested rows cleanly, stop and report, and use the smallest strongly validated shape rather than weakening validation.
  - **The status line,** top centre:
    - Waiting: `Waiting for players — 1/2`
    - Countdown: `Match starts in 12`, never counting up
    - Running and Alive: `3 alive · 6:42 left`
    - Running in the Hub: `Match in progress — 3 alive`, with **Spectate Match**
    - Hidden in Results
  - **The feed,** top right: `A eliminated B · 2 left`, or `B was eliminated · 2 left`. Each line lasts about 4 s, with at most 4 lines, and falls back safely for players who have left.
  - **The banner,** for 3 s: `VICTORY`, `<name> wins`, `Time out — no winner` or `No survivors`. The results panel follows, with `Returning to Hub in 8`.
  - A client that joins during Results may miss the rows.
  - **The elimination drop** gets a 6-stud burst for 1.5 s, and a beam easing from 0.3 to 0.7 transparency over 2 s, with no sound. The values go in `Tuning.match.display`.
  - **Out of scope:** Match sounds, music, streaks, a minimap, a live scoreboard, a final results screen and Match history.
  - **Who builds what.** The main session builds `Protocol`, `Match`, `MatchRules`, the gates and `ReturnToHub`. `studio-implementer` builds, from exact briefs, the Hub geometry, the HUD, feed, banner and results, `Spectating`'s presentation and the drop flash. The main session reviews it.
- **Specs.**
  - **New `MatchRulesSpec`:** the minimums, the phases and the cancelled countdown, the roster, `assignSpawns`, `eliminate`, every `checkEnd` case including the grace and precedence, the three gates across every phase and state, the end of Results, history surviving Return to Hub, and places and outcomes.
  - **New `MatchSpec`,** with fakes: the order of the start, elimination, leave, Return to Hub, grace and Results-end steps, administrative removals never eliminating, and Reference mode installing nothing.
  - **Also:**
    - `SessionRulesSpec`: `matchStats`, and the snapshot not being mutated.
    - `PickupsSpec`: `clearAll`, `placeMarkers` and clearing every kind of Pickup.
    - Kits: `takeAll` and `resetForMatch`.
    - `ProtocolSpec`: the three new remotes.
    - Config sanity: `minPlayersInStudio <= minPlayers`, and `0 < endResolutionSeconds < 0.5`.
- **Play checks,** run by `studio-tester`, checking rendering before every group of input or visual checks:
  - **A, Reference solo:** the Match stays dormant. There are no attributes and no Hub, the dummies are present, a death respawns on an arena pad after about 3 s, Q then E and a Charge, and a forged `ReturnToHub` is refused.
  - **B, Empty solo,** once after 17A and again after 17B:
    - the Hub, with `NotInMatch` for a forged `CastRequest`
    - the countdown and start: a fresh arena spawn, six Pickups and no dummies
    - collection
    - elimination: the drops and their flash, the body for 3 s, `NoSurvivors`, the results and the Hub
    - the next cycle
    - a reset eliminating
    - `ReturnToHub` accepted during Results and refused while Alive
    - a Timeout, with a TEMPORARY `maxSeconds` of about 30
  - **C, two clients:**
    - a real kill: the feed, the drop, the body then the killer in Spectate, `VICTORY` and `<name> wins`, the rows, and the Hub
    - Return to Hub, then **Spectate Match**, and the camera returning when the Match ends
    - a leave: `Left` exactly once, the drop, the grace, then `Winner`
    - a Timeout with two survivors
    - trades through the normal Combat path: both within 0.25 s gives `NoSurvivors`, and 0.4 s apart gives `Winner` with the late damage refused
  - **D, three clients, narrow:** one roster, cycling with Q, E and the buttons with wrapping, moving on automatically, `2 left`, a `Winner`, and a spectator who can't affect anything.
  - **TEMPORARY aids:** a damage and kill hook through Combat, and a lowered `maxSeconds`. They're removed afterwards and confirmed with a search, and never committed.
- **Done when** 17A and 17B both pass, Runs A–D all pass, the temporary aids are gone, and the docs and record are updated.

**Built and tested 2026-11-08 to 2026-11-09. All checks pass.** It stayed one ticket: 17A wasn't materially larger or riskier, so 17B was built straight after it.

- **Server side:**
  - The new pure `MatchRules` holds the phases, roster, eliminations, end, the gates, Return to Hub and the result rows.
  - The new `Match` system runs it, installs the policy and gates, and handles `ReturnToHub`.
  - `Characters` gained areas, the spawn policy, the target filter, `onDeath`, `spawnInArea`, `removeCharacter` and `lastPosition`.
  - `Kits` gained `takeAll`, `resetForMatch` and `onPlayerLeaving`. `Pickups` gained `clearAll`, `placeMarkers`, `drop`, `findFloor`, the `fromElimination` mark and the `canCollect` gate. `Casts` gained the `canAct` gate. `Session` gained `killerOf`, the Match snapshot and the Match log lines. `SessionRules` gained `snapshot` and `matchStats`.
  - `Dummies` returns early with an Empty Kit.
  - `Protocol` gained the three remotes, the strongly validated `resultRows` type and the Match attributes. `Tuning.match` holds the settled values and its `display` table.
- **Fixes made during review and testing:**
  - **Session record on leave.** `Session` now forgets a leaving player only after the other `PlayerRemoving` handlers have run, so a `Left` elimination still reads their Match numbers, whatever order the handlers fire in.
  - **Order at the end.** `Match` publishes the phase attributes before the Session log and `MatchEnded`, as settled.
  - **Kit lost on leave.** `Kits.takeAll` now takes the Kit held for the life that ended, even when the character has already gone. Before, a leave dropped nothing, because `ensure` saw the missing character as a new one and handed over a fresh empty Kit. Run D found it, and `KitsSpec` covers it now.
- **Client side,** built by `studio-implementer` from an exact brief and reviewed before acceptance:
  - The new `MatchHud` shows the status line, the feed, the elimination title, the spectate bar, the end banner and the results, and dims the slots.
  - The new `Spectating` drives the camera through `Camera.setSuspended`, and owns Q and E through `Input.setSlotOverride`.
  - `Input` passes no slot, Basic Attack or replacement presses on unless the player is Running and Alive.
  - `Feedback` skips the respawn-style "Killed by" notice in a Match.
  - `PickupVisuals` gives elimination drops their 6-stud flash and the beam that eases from 0.3 to 0.7. `CosmeticProjectiles.burst` takes an optional duration for it.
- **Specs:** 302 passed, 0 failed, run on the Server in Play. They include the new `MatchRulesSpec` (21, with the two Config checks), `MatchSpec` (13, on fakes around a clone of `Match`) and `KitsSpec` (3). The additions to `ProtocolSpec`, `SessionRulesSpec`, `KitRulesSpec` and `PickupsSpec` add 16.
- **Test setup:** a TEMPORARY `TEMPORARYMatchHook` script, with `hit` through `Combat.applyHit` and `grant` through `Kits.grant`, plus a TEMPORARY `maxSeconds` of 30 for the solo runs and 90 for the multi-client runs. All were removed after the runs. A search for `TEMPORARY` finds nothing, Tuning is back to `startingKit = "Reference"` and `maxSeconds = 480`, and the specs passed 302 of 302 again afterwards.

**A, Reference Kit, solo** (run after 17A and again after 17B):

| # | Check | Result |
|---|---|---|
| A1 | Specs | 301 passed, then 302 after the `takeAll` fix |
| A2 | Dormant | No Match attributes or `MatchState`. Spawned on an arena pad with 4 dummies and the Reference Kit. Every `MatchHud` element hidden and the slots not dimmed |
| A3 | Q, E and a Charge | No `CastRejected`, and `ChargingRank` 1 during the hold |
| A4 | Death | Respawned on an arena pad in 3.03 s with the Reference Kit, and the "Killed by" notice showed as before. No `Eliminated` line |
| A5 | Forged `ReturnToHub {}` | Nothing happened |
| A6 | Console | No errors |

**B, Empty Kit, solo** (17A, then again with 17B):

| # | Check | Result |
|---|---|---|
| B1 | Hub | `MatchState` Hub, in the Hub with an empty Kit, no dummies and no Pickups. `Match starts in N` counted down, and the slots were dimmed. A forged `CastRequest` got `NotInMatch`, a forged `ReplaceRequest` got the `NotInMatch` warning, and the hook found `NoTarget`. Real Q, E and clicks sent nothing |
| B2 | Start | Running with `MatchId` 1, a new character on an arena pad at full health, an empty Kit, exactly 6 Pickups and `MatchStart match=1 players=1`. `1 alive · 7:50 left`, with the slots not dimmed |
| B3 | Collection | Arc Burst and Tidal Bolt collected, and a Q tap was accepted |
| B4 | Elimination | `Eliminated ... cause=Died by=none alive=0 dropped=ArcBurst:1,TidalBolt:1`. The two drops landed 1.5 studs either side of the death spot, flagged. The beam eased 0.35 → 0.70 over 2.2 s, with a 6-stud burst. Then `NoSurvivors` at once, `No survivors` for 3 s, and Results with one row (`—`, `Eliminated`, Pickups `2/0/0`). The camera stayed on the body for about 3.2 s, then held still. The mouse was free, with `0 alive` and Return to Hub |
| B5 | Results end | Back in the Hub with a fresh character, an empty Kit and the Pickups cleared. The game camera returned, with the mouse locked. No extra `Eliminated` line |
| B6 | Next cycle | `MatchId` 2, a fresh arena spawn and 6 Pickups |
| B7 | Return to Hub while Alive | `refused, Alive`, with nothing changed |
| B8 | A reset | `Eliminated ... cause=Died by=none`, then `NoSurvivors` |
| B9 | Return to Hub during Results | A real click on the button put the player in the Hub with a new character, about 5 s before Results ended, with `ReturnToHub match=3`. A second request got `refused, InHub`, and `{x=1}` was malformed. The player wasn't respawned again at the end |
| B10 | Hub death in the Countdown | A Hub respawn in 3.1 s with no `Eliminated` line. The Countdown cancelled and restarted in full |
| B11 | Timeout (`maxSeconds` 30) | The status counted down to `0:00 left`. Then `Time out — no winner`, and Results with the row `Survived`, place `—`. Back to the Hub |
| B12 | Consoles | No errors |

**C, two clients:**

| # | Check | Result |
|---|---|---|
| C1 | A real kill | `Kill`, then `Eliminated ... by=player:-1 alive=1`, then `MatchEnd cause=Winner` 0.26–0.28 s later. Both feeds read `Player1 eliminated Player2 · 1 left`. The victim saw `Eliminated by Player1`, the camera on their own body until +3 s and then on the killer with `Spectating Player1 · 1 alive`, and `Player1 wins`. The winner saw `VICTORY`. Rows: 1st `Winner` with 1 kill, and 2nd `Eliminated by Player1`. Both returned to the Hub with the game camera, and the Pickups were cleared |
| C2 | A leave (kick) | `Eliminated ... cause=Left by=none alive=1` exactly once, then `Winner`. The other client saw `Player2 was eliminated · 1 left` and `VICTORY`, and the leaver's row was present |
| C3 | Trades | Two hits 0.08 s apart: both Eliminated, then `NoSurvivors` with no places. Hits 0.4 s apart: the late one found `NoTarget`, the winner's health was unchanged, and the result was `Winner` |
| C4 | Timeout with two survivors (`maxSeconds` 90) | `MatchEnd cause=Timeout winner=none alive=2`, with both rows `Survived`, place `—` |

**D, three clients:**

| # | Check | Result |
|---|---|---|
| D1 | One roster, a kill | `MatchStart players=3`. `Eliminated ... alive=2 dropped=ArcBurst:1,TidalBolt:1`, with the drops 1.5 studs either side. Every feed read `· 2 left`. The victim watched their body, then the killer, with `Spectating Player1 · 2 alive` |
| D2 | A spectator can't affect anything | A `CastRequest` got `NotInMatch`, a `ReplaceRequest` got the `NotInMatch` warning, and the hook found `NoTarget`. Real Q and E changed the target and sent no Cast |
| D3 | Cycling | E, E, Q, ▶ and ◀ wrapped by UserId, with real button clicks |
| D4 | Moving on automatically | When the watched player was eliminated, the camera moved to the last one alive within 0.12 s. Rows: 1st, 2nd, 3rd in reverse order of elimination |
| D5 | Return to Hub, then Spectate Match | Real clicks. From the Hub: `Match in progress — 2 alive`, then Spectate Match onto the first alive player, with `Back` and the character staying in the Hub. When Results began, the spectator dropped out within 0.12 s, back to the game camera. `Back` worked in a later Match |
| D6 | A leave with a Kit | It failed first with `dropped=none`, which found the `takeAll` bug. Re-run after the fix: `Eliminated ... cause=Left by=none alive=1 dropped=ArcBurst:2`, the flagged drop at the last position, then `Winner` |
| D7 | Spectate Match by key | After the fix below, a real G press from the Hub spectated the first alive player, and a second G backed out. G did nothing for an Alive player |
| D8 | A fall out of the world with a Kit | `dropped=TidalBolt:2` at the last floor position, flagged. The cause was `Died`, not `Removed` (see below) |

**Settled during testing (2026-11-09):** the Hub's mouse is locked to the centre, so a real player couldn't reach the Spectate Match button. The tester's clicks only landed because the MCP moves the cursor to the button directly. The developer chose a key: **G** toggles Hub spectating, and the buttons read `Spectate Match (G)` and `Back (G)`.

**Accepted with notes:**

- **`PlayerEliminated.killerId`** is optional and left out when there was no killer, rather than an empty string. This matches every other optional field in `Protocol`. A `shortString` can't be empty, so the empty-string version would have needed its own case.
- **A row is final at its elimination.** In a trade, the second hit's kill is in the session log, but the player who made it had already been eliminated, so their row shows the kills they had then.
- **A fall below the world** reports `Died`, because the engine kills the Humanoid before destroying the character. That's covered by the settled rule that `Died` covers falls. The `Removed` path stays for a character destroyed without dying, and `MatchSpec` covers it.
- **A leaver's row** reads `Eliminated` with place and cause `Left`, and the panel doesn't spell out "Left". That's allowed by "Left may show as such".
- **A Return to Hub during Results** leaves the results panel on screen in the Hub until Results end. Results go to everyone, and the panel doesn't block movement.
- **`Waiting`** shows for about 0.5 s after Results and after a Hub death, before the Countdown restarts.
- **The body view** during the first 3 s looks steeply down at the body. It's a starting value.
- **The body's removal** at 3 s was bracketed in Play (there at 1.3 s, gone by 6.4 s) and is pinned on fakes in `MatchSpec`.

**Published by the developer on 2026-11-09 as version 70**, recorded in the checkpoints above.

### 18. Match map, Caches and Sentries

- **Scope:**
  - The roughly 250 × 250 greybox and its spawn points.
  - Breakable Rift Caches with crack stages.
  - Stationary Sentries with their Telegraph, kept thin on the real Tidal Bolt path, with no moving monster system.
  - The shuffled Ability bag, the early-variety adjustment, and central Caches' two drops set apart.
- **Starts after** the Match lifecycle works in the existing arena.

**Settled 2026-11-09, before building.**

- **The map.**
  - **`Workspace.MatchMap`** is centred at about (0, 0, −450): a greybox floor of about 250 × 250 at y = 0, with perimeter walls about 20 studs high and no route to the arena or the Hub. The size is starting tuning, not a fixed dimension.
  - `Workspace.Arena` stays unchanged for the Reference Kit and Stage 3, and `Workspace.Hub` stays as it is.
  - **Spawns.** Six pads on a ring of about r = 105, in a new `"Map"` spawn area rather than `"Arena"`. In an Empty-Kit Match, `Match` deals them through the existing `MatchRules.assignSpawns`, so 2–6 players never share a pad, and Ticket 17's four-pad wrapping no longer applies at the intended player counts.
  - **A risk gradient, by ring:**
    - the edge: the six pads
    - the outer ring: 10 unguarded Caches, reachable early from the spawns
    - the middle: scattered cover at about r = 50, breaking long sight lines
    - the centre: a solid core of about r = 10 with three radial walls making three bays. Each bay has one central Cache toward the front and one Sentry behind it on a low pedestal. The walls stop all three Sentries acquiring the same player, so a player committing to one central Cache normally faces one Sentry.
  - **10 outer Caches, not 12,** so a Match still holds exactly 16 Ability drops (10 × 1 + 3 × 2) and four Abilities can have exactly four copies each. Pads don't need two Caches of their own: each has about two reasonable early routes, and some Caches are shared between neighbouring pads, which may give light early contest. A first check, with Caches every 36° on r ≈ 80, put every pad within about 27–54 studs of two Caches, with the Caches at 90° and 270° shared. If the coordinate pass can't give every pad reasonable early access, report it before changing the loot total. The equal bag is never broken silently.
  - **Central value** is two drops, a choice between them, Sentry risk and PvP convergence. No higher-Rank central loot.
  - **Ticket 16's placement path is removed:** the six `PickupPlacements` markers, `Pickups.placeMarkers` and its spec. Caches are the source of world Ability loot, and replacement and elimination drops stay normal loose Pickups.
- **Environment roots.**
  - `Workspace.Arena` was hardcoded as the only environment in nine places: server cover, the client's aim cover check, camera collision, cosmetic projectile and Sector impacts, the dummy's sight line, the floor raycasts in `Pickups` and `Match`, and `Characters`' areas.
  - One small neutral shared module, `ReplicatedStorage.Shared.Environment`, is the single source of the folders that are static environment geometry: `Arena`, `MatchMap` and `Hub`. It sits beside `Config`, `Rules` and `Net`, not in `Protocol`, which describes network contracts. Every environment query uses it, and the list isn't repeated at call sites. It isn't a world-query framework.
  - Having `MatchMap` and `Hub` in the list changes nothing in the arena, because no ray from the arena can reach them.
- **Rift Caches.**
  - **In code,** a Cache is a registered combat target: an anchored greybox model with a hidden Humanoid for health, a `HumanoidRootPart` and the existing fixed Hitbox. The visible Cache is sized to match the Hitbox, about 3.4 × 5 × 3.4.
  - Runtime Caches live in `Workspace.Caches`, never in the environment roots. Their markers, `CacheOuter01`–`10` and `CacheCentral1`–`3`, are invisible, non-colliding, non-queryable setup data in `MatchMap`.
  - **A damageable object, not a combat actor.** Player attacks damage it at the attacker's authoritative Rank: the Basic Attack, Tidal Bolt, Arc Burst, and later Rift Hook if its projectile should normally damage objects. A Cache hit takes no part in Statuses, Reactions, Knockback, Evade, Kill Recovery, player fight tracking, damage-dealt Match stats or Reaction counters, so a Wet Cache hit by Arc Burst can't make a fake Reaction. It's a narrow extension of the existing Combat path, not a separate Cache combat implementation.
  - **Who can damage one.** Only a player who is Alive while the Match is Running, **checked at impact**. So an attack still in flight from a player eliminated since doesn't damage a Cache, though Ticket 17 lets such attacks still hit players. Sentry attacks never damage a Cache.
  - **Feedback.** The attacker gets the normal hit confirm, with no kill marker when the Cache breaks. The break has its own world feedback and log: `CacheBroken match= cache= kind= by= ability= seconds= drops=`.
  - **Health,** as tuning: **outer 75**, about 5 s of Basic Attack alone, and **central 120**, about 8 s.
  - **A known tuning hypothesis:** spawns are about 2–3.5 s from their nearest Caches, so a solo player's first Ability is expected at about 8–10 s, and with only two Abilities and early variety a full Kit at about 25 s. That's well ahead of the 30–45 s and 1.5–2 minute targets. Health isn't raised to force the old targets before the map has been played. Ticket 20's measurements decide whether Cache health, distance, number or placement change. What matters here: opening loot takes commitment, melee exposes the player, and a central Cache costs materially more time than an outer one.
  - **Readability.** `CacheStage` is 0 when intact, 1 below about 66% and 2 below about 33%, plus `CacheBroken`. Clients draw a darker, damaged look with glowing crack lines that grow. No health bar. Every unopened Cache looks the same, and its contents never show in attributes or visuals. Being central and guarded may signal risk, but never what's inside.
  - **The break.** A visible burst and a loud sound. The Cache stops being a target, and a low, non-colliding stump stays for the rest of the Match, only to show it's been opened. The stump is neither a target nor meaningful cover.
  - **Cover.** An unopened Cache intercepts projectiles that hit it, whoever fired them. A projectile stops there, damaging the Cache if the attacker may, and never continues to a player behind. This comes from the target and projectile interaction, not from putting Caches in the environment roots.
  - **Drops,** normal Rank 1 Pickups through `Pickups.spawn`, placed with the established floor logic:
    - **Outer:** one, at or near the broken Cache.
    - **Central:** two different Abilities, either side of the Cache across the bay's direction, at `centralDropSpreadStuds = 5` each side, about 10 studs apart. At 8 apart, the midpoint would sit exactly 4 studs from both, the collection radius, so a boundary case could collect both. Breaking a central Cache should give a real spatial choice, with no selection UI.
  - **Lifecycle.**
    - Match start: all Caches built fresh, intact, with their hidden contents assigned.
    - Running: they can be damaged and broken, and none comes back.
    - Results: the models stay for continuity, but the gates are closed.
    - Results end: removed as part of the Match clean-up.
    - Waiting, the Countdown and the Reference Kit: none exist.
- **Sentries.**
  - **In code,** a Sentry is a stationary registered target: an anchored greybox body with a hidden Humanoid, a `HumanoidRootPart`, the fixed Hitbox, a silhouette clearly unlike a Cache, and a visible firing point near the top.
  - Runtime Sentries live in `Workspace.Sentries`, and their markers `Sentry1`–`3` belong to `MatchMap`.
  - One small `Sentries` server system drives all three from one loop at about 5 Hz. No pathfinding, patrol, Threat, Home Territory, Monster Families, Elites or scheduler framework.
  - **Targeting.** The nearest valid player, Alive in a Running Match, within about **45 studs**, with a clear authoritative sight line through the environment roots. Caches and Sentries don't need to be opaque to acquisition. Fired projectiles still follow the normal collision rules below.
  - **The cycle.**
    - Acquire a target, Telegraph for about **0.5 s**, then fire.
    - Recover for about **2.5 s** before the next Telegraph, so shots come about **3 s** apart.
    - With no valid target, idle.
    - If the target becomes invalid or goes out of sight before the shot, cancel it and return to idle. It never fires without a full Telegraph.
  - **The aim locks when the Telegraph starts.**
    - The Sentry records an authoritative aim point from the target's Hitbox then, Telegraphs toward it, and fires at that point.
    - The target can avoid it by moving, sidestepping, Dashing or breaking line of sight. That's intended: readable PvE pressure, not unavoidable attrition.
    - If a locked aim proves too easy to avoid, a visible tracking Telegraph or a later lock can be tested then.
  - **The Telegraph.**
    - The server sets `SentryTelegraphEndsAt` in server time, and clients draw a growing orb in Tidal Bolt's colour at the firing point, related to its Charging look.
    - It isn't real Charging: no Charge percentage, scaling, interruption rules or full-Charge behaviour.
    - The bolt starts at that visible firing point.
  - **The shot.** A Rank 1 tap Tidal Bolt through `Casts.castFromServer`: about 12 damage, Wet, the normal ~25 Knockback, which ends a Charge, and normal authoritative projectile behaviour. No Sentry-specific damage path.
  - **Caches block Sentry bolts too.** A bolt never passes through an intact Cache. Each bay's Sentry, firing point and Cache are placed so the guard has a useful line on a player breaking that Cache without passing through it: a sideways offset, a raised firing point, a moved Cache, or changes to the bay. If the first layout can't do that cleanly, the greybox changes, never the collision rules for Sentries.
  - **Health: 80,** as tuning, so killing the guard is a real but optional commitment.
  - **Combat.** A Sentry is a real actor in the shared interaction system: it takes normal damage, can be made Wet, and Reactions can trigger on it, along with future Statuses. It doesn't Evade or recover health naturally, and Knockback doesn't move it, with no special forced movement.
  - **Killing one** gives **no Kill Recovery**, which stays a player-kill reward, so PvE can't become a healing source. It drops nothing, so all world loot stays in the 16-drop bag.
  - **Stats.** Hits, kills and Reactions on a Sentry don't count toward PvP kills, PvP damage, player fights or Match-result Reactions. They're logged on their own: `SentryKilled match= sentry= by= ability= seconds=`. A Sentry's hits on players stay real combat damage, with Wet and Knockback, which is the intended PvE → PvP interaction.
  - **Death.** It becomes a dark wreck that can't be targeted, with no more Telegraphs or shots, for the rest of the Match.
  - **Lifecycle.** Built fresh at Match start and cleared when Results end. None exist while Waiting, in the Countdown, or with the Reference Kit.
  - **A player eliminated by a Sentry** sees the generic `Eliminated`. `PlayerEliminated` isn't widened into a non-player killer model. The server log names the source, `Eliminated ... cause=Died by=sentry:2`, to answer whether Sentries end too many runs. It never counts as a player kill.
- **The Ability bag and early variety.**
  - **`Tuning.loot.pool`** is `TidalBolt` and `ArcBurst` in this ticket, and Ticket 19 adds `RiftHook` and `StoneGuard`. The total is 16 world drops, all at Rank 1.
  - **The new pure `LootRules`:**
    - **`buildBag(pool, total)`** makes equal copies: 8 / 8 now, and 4 / 4 / 4 / 4 after Ticket 19. Equal counts are a strict invariant.
    - **`nearestPairs(pads, outerCaches)`** finds each pad's two nearest outer Caches from positions, with ties broken by Cache name. There are no hand-kept pairing tables, so moving a marker updates early variety by itself.
    - **`deal(bag, caches, pairs, random)`**:
      1. Shuffle with `Match`'s injected random source, with no other randomness, so tests and Matches are reproducible.
      2. Deal the central Caches first, each getting two **different** Abilities, never a duplicate just because it was next. With two Abilities, every central Cache gets one Tidal Bolt and one Arc Burst.
      3. Deal the rest across the 10 outer Caches, then run a bounded, deterministic repair. Where a pad's two nearest Caches match, it tries a swap with another outer Cache, accepted only if it fixes that pair without breaking a valid pair. It's not a general constraint solver.
  - **Equal counts are strict, and early variety is a best effort.** If the repair can't fix every pair, the equal deal stands, the Match starts, and the log says `variety=partial`.
  - **When the invariant breaks:**
    - Specs and config checks fail loudly in development.
    - At runtime, `Match` checks the pool and total once at server start. If they don't divide evenly, it logs one high-severity error, for example `[Riftfall] Matches disabled: loot pool 3 doesn't divide 16`. It stays in Waiting for the rest of that server session, with no Countdown, retries or repeated logs, and players stay safely in the Hub.
    - A defensive check also runs when dealing, before anything in the world changes. If it fails, the start is cancelled before any Cache, Sentry or loot exists, `Match` stays in Waiting, and Matches are disabled the same way, logged once.
    - An unequal fallback bag is never dealt.
  - **Hidden contents.** Each Cache's contents live only in its server record, as a generic list of `{ abilityId, rank }`. Nothing replicates until the break makes real Pickups. Weighted Drop Tables, rarity, and event or Elite loot can later replace `buildBag` and the equal deal without changing how Caches hold loot. No weights now.
  - **Logs.**
    - **`LootDeal match=3 total=16 counts=ArcBurst:8,TidalBolt:8 variety=ok`** is one line per Match, with names in a fixed order.
    - **`LootContents match=3 ...`** is a developer-only line listing each Cache's contents, for Play checks, debugging early variety and pacing analysis. It's never sent to clients. If it grows too long with more content, it moves behind a debug flag.
- **Ownership.**
  - **New pure Rules, with specs.** Line of sight and other world queries are passed in as functions, following `CastRules.originFor`.
    - `LootRules`, as above.
    - `CacheRules`: the stage from health, the central drop spots, whether an attack may damage the Cache, and other deterministic Cache decisions.
    - `SentryRules`: the Idle → Telegraph → Recover cycle as plain data. It covers picking the nearest eligible target, starting the Telegraph, locking the aim, cancelling, when to fire and the cadence. It doesn't become a monster AI framework.
  - **The new `Caches` server system** owns the runtime records and their hidden loot, building them from the markers, damage and health, `CacheStage` and `CacheBroken`, the drops, and the `LootDeal`, `LootContents` and `CacheBroken` logs. Its lifecycle is `setup(random, ...)` and `clear()`.
  - **The new `Sentries` server system** owns the three records, acquiring targets, Telegraph timing, the locked aim, firing and cancelling, health and death, and the `SentryKilled` log. Its lifecycle is `setup()` and `clear()`.
  - **Target kinds.**
    - Registered targets gain an explicit `kind`: `player`, `dummy`, `sentry` or `cache`. It's never inferred from names or folders.
    - **`Combat.applyHit`** has one narrow early branch for a Cache. It goes to the Cache's handler, which applies valid damage only, then sends the normal Hit result and stops. The Combat pipeline isn't forked.
    - Sentries take the normal path. Their Knockback handler moves nothing, and they give no Kill Recovery.
  - **`Session`** leaves Cache and Sentry targets out of player fights, PvP damage, PvP kills and Match-result Reactions, with dummies unchanged. It records a Sentry killer for the `Eliminated` log only. `Match` passes `PlayerEliminated`, the rows and Spectate a killer only when it's a player.
  - **`Casts.castFromServer`** gains an optional `options.origin`, trusted because only server code calls it. Sentries use it so the bolt starts at the Telegraph's firing point. No client payload carries it, player Casts keep the hand-origin rules, and all other validation and scaling stay as they are.
  - **`Match`.**
    - **Start, in this order:**
      1. clear loose Pickups
      2. check and deal the loot
      3. `Caches.setup(random)`
      4. `Sentries.setup()`
      5. spawn the players on the `"Map"` pads
    - Players never become active in a partly built PvE world.
    - **The end of Results** clears the Caches, Sentries and Pickups before returning to Waiting.
    - `placeMarkers` goes, and `Match`'s fallback spawn and grounded-position raycasts move to the map.
  - **Clients.** A new `PveVisuals` draws the crack stages, the break burst and sound, the stump, the Sentry Telegraph orb and the wreck, from tags and attributes only. No new remotes, unless building shows that an attribute can't carry a particular transient effect cleanly.
  - **Dormant with the Reference Kit.** `Match` returns early as before. So there's no Cache or Sentry setup, no loot deal, no Cache or Sentry logs, no Sentry loop doing work, no use of the map's pads and no world Ability loot. `MatchMap` and the Hub may sit in the workspace, and the `kind` branches leave players and dummies on their current paths. Ticket 17's Run A is repeated as the regression check.
  - **Who builds what.**
    - **The main session** builds the Rules, `Caches`, `Sentries`, the `Combat`, `Session`, `Casts` and `Match` changes, the environment roots, and the exact coordinate list.
    - **`studio-implementer`** builds the `MatchMap` geometry and markers, and `PveVisuals`, from exact briefs.
    - The main session reviews both before accepting them.
- **Specs.**
  - **The new `LootRulesSpec`:**
    - **`buildBag`:** 8 / 8, and 4 / 4 / 4 / 4 using the future ids as plain strings, with an uneven split refused.
    - **`nearestPairs`:** the real coordinates, deterministic order, and ties broken by name.
    - **`deal`,** over about 200 seeds for both pools:
      - the counts stay exact, and nothing is invented or lost
      - every central Cache gets two different Abilities
      - every bag entry goes to exactly one Cache, and every Cache gets the right number
      - the repair ends
      - `variety` matches the actual result
      - the same seed and inputs give the same deal
    - Nothing depends on the shuffle order beyond its determinism.
  - **The new `CacheRulesSpec`:**
    - the stages either side of 66% and of 33%, so `<` against `<=` is pinned
    - the central spots: ±5 across the given vector, about 10 apart and symmetric about the Cache
    - **the attacker gate:** it allows an Alive player while Running. It refuses the same player once eliminated, a Sentry, anything outside Running, and other kinds, using authoritative state and kind, never names
  - **The new `SentryRulesSpec`.** Assertions use the configured values rather than fuzzy wall-clock timing.
    - **Targeting:** the nearest visible valid player wins, and players beyond 45 studs, out of sight or not Alive are skipped. Equal distances resolve deterministically.
    - **The locked aim:**
      - the point is taken when the Telegraph starts, and later movement doesn't change it
      - no shot comes before the full Telegraph, and the shot goes to the locked point after it
    - **Cancelling:**
      - an invalid target or lost sight cancels the shot, with no projectile
      - the next shot needs a fresh full Telegraph
    - **Cadence:**
      - a 2.5 s recovery and about 3 s between shots
      - no double shot, and no shot from Idle or while Recovering
      - a dead Sentry never starts another cycle
  - **The new `CachesSpec`, on fakes:**
    - **Setup:** 13 Caches, 10 outer and 3 central, with server-side loot records and no attribute exposing contents.
    - **Damage:**
      - valid player attacks lower health, and the stages change at the thresholds
      - the Cache breaks once, and later hits do nothing
      - an outer Cache makes one Pickup, and a central one makes two at ±5
    - **When broken:** the Cache is no longer a target, and the stump is presentation only.
    - **Refused:** any attack outside Running, and Sentry attacks. **Refusing a Sentry's damage doesn't mean its projectile passes through.** Collision belongs to the projectile path, and whether it does damage to `CacheRules` and `Combat`. Play checks this too.
    - **Clean-up:** `clear` removes everything, the break log appears once, and contents never reach attributes.
  - **The new `SentriesSpec`, on fakes:**
    - setup makes all three, and the Telegraph attribute follows the authoritative cycle
    - **`castFromServer`** gets the trusted firing origin, the locked aim and a Rank 1 Tidal Bolt
    - a cancelled Telegraph makes no Cast
    - death stops that Sentry's activity and leaves a wreck that can't be targeted, and `clear` resets everything
    - its Casts can't target or damage another Sentry
  - **The new `CombatSpec`, on fakes:**
    - **A Cache hit** gives valid damage and the Hit result only: no Statuses, Reactions, Knockback, Evade, Kill Recovery or PvP Session accounting. A Sentry-fired hit that reaches a Cache is refused damage, with nothing else happening either.
    - **A Sentry hit:** normal damage, Wet and Reactions, with no displacement, no Kill Recovery and no PvP counters.
    - **Players and dummies:** unchanged-path cases, without rewriting existing assertions.
  - **The new `MatchMapSpec`, in Play against the real map:**
    - **The markers:** 6 Map pads, 10 outer and 3 central Cache markers, and 3 Sentry markers, all inside the walls.
    - **The routes:** each pad has two nearby outer Caches, consistent with the layout.
    - **Out of Sentry range:** every pad and outer Cache lies beyond 45 studs of a Sentry.
    - **For each bay,** at the representative spots where a player stands to break its Cache:
      - its Sentry has a clear line of fire past its intact Cache's Hitbox
      - the bay's static geometry doesn't block that line
      - the other two Sentries can't see those spots
    - It checks the positions that matter for play, without trying to prove a Sentry can never see any point in another bay.
  - **`MatchSpec`, updated:**
    - **The start order above,** with nobody active in a partly built PvE world.
    - **Results end:** the Caches, Sentries and Pickups are cleared, and the next Match gets fresh PvE.
    - **Bad config at startup:** one error, Matches disabled, Waiting kept, no Countdown and no repeated errors.
    - **A failure at deal time:** it fails before anything in the world changes, with no partial Caches or Sentries and no spawns. `Match` stays safely in Waiting, and Matches are disabled.
  - **`SessionRulesSpec`:**
    - Cache and Sentry targets don't add to PvP damage, kills, fights or Match-result Reactions, and dummies behave exactly as now.
    - A Sentry's Wet on a player doesn't count, by itself, as any player's Reaction or damage.
  - **`PickupsSpec`:** the `placeMarkers` test goes. Cache drops, replacement drops, elimination drops and `clearAll` stay on the one normal Pickup path, with no second Pickup type.
  - **Config checks:**
    - the pool isn't empty, the total is above 0, and the pool divides the total evenly
    - the central spread per side is more than the collection radius, so the two collection spheres can't touch at the midpoint: `2 × spread > 2 × collectRadius`, which reduces to `spread > collectRadius`
    - the Telegraph is above 0 and shorter than the full cadence
    - sight range, Cache health and Sentry health are above 0
    - every environment folder resolves
- **Play checks,** run by `studio-tester`, checking rendering and state before each group of input:
  - **A, Reference solo:** Ticket 17's Run A again, with no Match lifecycle, runtime Caches, Sentries or `LootDeal`. The dummies are there, and arena respawns, Q, E, Charging, cover, camera collision and projectile and Sector impacts all behave as before. The distant `MatchMap` and Hub make no visible difference in the arena.
  - **B, Empty solo:**
    - **B1, Match start:** a `"Map"` spawn, 13 Caches, 3 Sentries, and no loose Pickups before a Cache breaks. `LootDeal` shows `ArcBurst:8,TidalBolt:8` and the expected variety, and no Cache attribute holds its Abilities.
    - **B2, an outer Cache with the Basic Attack:**
      - the stages appear, and it breaks at 75 after about 5 s of uninterrupted attacks
      - ordinary hit confirms and no kill marker
      - exactly one Pickup, the burst and sound, and a correct `CacheBroken`
      - the stump can't be hit and doesn't meaningfully block a projectile
      - **recorded as measurements, not a pass or fail:** the time from Running to the first Ability, and to the first full two-slot Kit
    - **B3, Tidal Bolt into a Cache:** it does damage and stops on the Cache. The Cache doesn't become Wet, with no Status or Reaction accounting.
    - **B4, a Sentry:**
      - **Timing:** a visible Telegraph of about 0.5 s, and shots about 3 s apart.
      - **The hit:** about 12 damage, Wet and Knockback, which ends a Charge.
      - **Dodging:** a clear sideways move during the Telegraph makes the shot go to the old point and miss.
      - **Breaking sight:** doing it before the Telegraph ends means no projectile.
      - **Cache cover, the explicit regression:** with the player behind an intact Cache from the Sentry, the bolt stops at the Cache. The Cache takes **no damage**, because Sentry attackers are refused, and the player takes none either. The bolt doesn't pass through. This proves collision and damage eligibility are separate.
    - **B5, a central Cache:**
      - two different Pickups about 10 studs apart
      - standing at the Cache's centre after the break collects neither
      - walking to one collects only that one, the other stays, and it can be collected later if the Kit allows
    - **B6, killing a Sentry:**
      - Wet, then Arc Burst, makes a real Reaction on it
      - it dies at the expected health, with no Kill Recovery and no loot
      - `SentryKilled`, the wreck, and no more Telegraphs or shots
      - no PvP kill, PvP damage or Match-result Reaction added
    - **B7, a Sentry eliminating the player:** the aid lowers the player's health, then a real Sentry bolt finishes them.
      - The server log has `by=sentry:N`.
      - The client shows the generic `Eliminated`, with no player killer.
      - Spectate doesn't try to follow a Sentry.
      - The whole-Kit drops land on the map's floor.
    - **B8, the next Match:** when Results end, the old Caches, Sentries and loose Pickups are gone. The next Match builds fresh Caches and Sentries with a fresh hidden deal, and no broken or dead PvE state carries over.
  - **C, two clients, narrow:**
    - **C1, PvE setting up a PvP Reaction.** A Sentry's Tidal Bolt hits A, so A is Wet with the **Sentry as its Applied By**. B hits A with Arc Burst, and the Lightning and Wet Reaction triggers.
      - **B is the Reaction's Triggered By** and gets the PvP Reaction credit.
      - The consumed Wet may still name the Sentry as Applied By.
      - The Sentry gets no player credit, and damage and results follow the existing Reaction rules.
      - These are two separate roles, and the record names both.
    - **C2, a Cache as PvP cover.** B's Tidal Bolt at A, with an intact Cache between them: the Cache intercepts it **and takes B's damage**, and A isn't hit. It's the counterpart to B4, where the Cache intercepts a Sentry bolt but can't be damaged by it.
    - **C3, results:** after real PvP, the results have only PvP counters. B's Reaction on A is valid PvP and counts, and PvE adds no kills, damage or Reactions.
  - **The TEMPORARY aid.**
    - **Its operations:** one server helper, `TEMPORARYMatchHook`, offering `hit` through `Combat.applyHit`, `grant` through `Kits.grant` and `setHealth`. It's for setup and control, never fake outcomes: a Sentry elimination comes from a real Sentry Cast, and Cache damage from real attacks where that's practical.
    - **Removing it:**
      - It's removed before completion, and never committed.
      - The proof is that no script named `TEMPORARYMatchHook` exists, and that no runtime code references the hook or its test-only operations.
      - This isn't a repository-wide search for `TEMPORARY`, because the docs may describe the procedure.
- **Done when:**
  - every new and updated spec passes
  - Runs A, B and C pass
  - the map matches the reviewed coordinate brief, and the bay line-of-fire checks pass
  - the `LootDeal` counts and variety match what's expected
  - the first-Ability and full-Kit times are recorded for Ticket 20
  - the aid is gone, and no production code references it
  - the record, `code-structure.md` and any new reusable quirk are written
  - the main session has reviewed the map and `PveVisuals`
- **No Ticket 19 work** starts automatically.

**Built and tested 2026-11-09. All checks pass.**

- **The map:**
  - The main session built `Workspace.MatchMap` straight from the coordinate list, a single script that was the brief itself, rather than through `studio-implementer`.
  - **The shell:** a 250 × 250 floor with its top at y = 0, perimeter walls 24 studs high (from y = −4 to 20), and pads `MapSpawn1`–`6` on r = 105, every 60° from 0°, facing the centre.
  - **Cover:** six cover walls (16 × 10 × 3), tangential at r = 50 on the pads' directions, and six low blocks (8 × 4 × 8) at r = 65 between them.
  - **The centre:** a cylinder core of r = 10 and 16 high, with radial walls (29 × 12 × 3) from r = 9 to r = 38 at 90°, 210° and 330°. They make bays centred on 30°, 150° and 270°.
  - **Outer Caches:** at r = 80, every 36° from 18°. Two are each shared by two pads: the one at 90°, by the pads at 60° and 120°, and the one at 270°, by the pads at 240° and 300°. Every pad has two within about 27–54 studs.
  - **Each bay:** the central Cache sits on the bay's axis at r = 26, facing outward. The Sentry stands 15 out and 13 across, on a 2-stud pedestal, with its root at y = 5.2 and its firing point 2.4 above that.
  - **Lines of fire.** A tap bolt clears the Cache's Hitbox (radius 1.5, plus the bolt's 0.6 and the 0.3 tolerance) by 2.9 studs at the front breaking spot. The Sentry-side and front-diagonal spots are open. The one spot it can't reach is straight past the Cache from the Sentry, the shadow every single Sentry leaves. B4 uses it as the cover check.
- **Server side:**
  - **New shared code:** `Environment` holds the one list of environment folders, and all nine former `Arena` call sites use it, including `Cover.new`, which now takes a list.
  - **New pure Rules:** `LootRules`, `CacheRules` and `SentryRules`.
  - **New systems:** `Caches`, with `prepare(random)` to deal without touching the world, `setup(deal, matchId)` and `clear()`, and `Sentries`, with `setup`, `clear` and `stepAll`. Each makes its workspace folder at the first setup, and the Sentry loop starts then too.
  - **`Characters`:** targets carry a `kind`, an optional `onHit` and `onKilled`, and `unregisterTarget`. The `"Map"` area reads `MatchMap`.
  - **`Combat`:** the Cache branch, and `Blocked` for refused Cache hits and for a Sentry's hit on another Sentry. A new `CombatResult` kind, `Blocked`, ends the Cosmetic Projectile there, and `Feedback` draws a plain impact for it. No new remote.
  - **`Recovery`** skips Caches and Sentries. **`Session`** applies `SessionRules.countsForPlayers` and remembers a Sentry's kill for the log. **`Casts.castFromServer`** takes `options.origin`.
  - **`Match`:**
    - It checks the bag at startup and deals just before a due start, using the new `MatchRules.isStartDue` and `disable`.
    - It sets Caches and Sentries up and clears them, deals spawns from the `"Map"` pads, and keeps Sentry killers out of player-facing data.
  - **Ticket 16's placement path is gone:** `Pickups.placeMarkers`, its spec and the `PickupPlacements` folder.
  - **Small additions for specs,** in the style of the existing `manual` options: `Combat` takes a `fireResult` dependency, and `Recovery.start` takes `manual`.
- **Client side,** built by `studio-implementer` from an exact brief and reviewed before acceptance:
  - The new `PveVisuals` draws the stages, the break burst and the `CacheBreak` sound (a built-in sound, checked to load), the stump colour, the Sentry orb and the wreck.
  - `Tuning.caches.display` and `Tuning.sentries.display` hold its values.
- **A fix during review:** the Cache models were first named `CacheCacheOuter01`, a doubled prefix the implementer found. They now take the marker's name.
- **Specs:** 356 passed, 0 failed, run on the Server in Play with both the Reference and the Empty Kit, and again in the two-client session.
  - **New specs:**
    - `LootRulesSpec` (9)
    - `CacheRulesSpec` (6)
    - `SentryRulesSpec` (8)
    - `CachesSpec` (7)
    - `SentriesSpec` (6)
    - `CombatSpec` (6)
    - `MatchMapSpec` (6)
  - **Additions:** `MatchSpec` gained 4 tests, `MatchRulesSpec` 2 and `SessionRulesSpec` 1. `PickupsSpec` lost its `placeMarkers` test.
- **Test setup:**
  - **The aid:** a TEMPORARY `TEMPORARYMatchHook` server script offered `hit` through `Combat.applyHit`, `grant` through `Kits.grant` and `setHealth`. Its `grant` first omitted the Rank and always returned false, so Run B built its Kits from real Caches instead.
  - **The Starting Kit:** it was `Empty` for Runs B and C.
  - **Afterwards:** the hook and its BindableFunction were deleted. No instance in the place has the hook's name, no script source mentions `TEMPORARY`, and `startingKit` is back to `"Reference"`.

**A, Reference Kit, solo,** plus the two rendering checks in the two-client session:

| # | Check | Result |
|---|---|---|
| A1 | Dormant | No Match attributes or `MatchState`, no `Caches` or `Sentries` folder, `PveVisuals` empty, and no `LootDeal`, `CacheBroken`, `SentryKilled` or `MatchStart` lines. The 4 dummies are there, the player is on an arena pad, and the Kit is Tidal Bolt / Arc Burst |
| A2 | Q, E and a Charge (forged, with rendering off) | No `CastRejected`. `ChargingRank` 1 during the hold |
| A3 | Dummy hits | Tidal Bolt did 12 and made the dummy Wet. Arc Burst then did 30 with `WetReaction` |
| A4 | Cover | A tap through `CoverCentre` reached nobody, and a clear shot hit for 12. With two clients, the other client drew the bolt's impact on `CoverNorthEast`'s face at z = −30 |
| A5 | Camera collision (two clients, rendering) | In the north-east corner the camera stopped at z = −49.5, inside the wall's face at −50, pulled in to 3.8 studs from 9 |
| A6 | Death | Respawned on an arena pad in 3.15 s with the Reference Kit. No `Eliminated` line |
| A7 | Forged `ReturnToHub {}` | Nothing changed |
| A8 | The ShootingDummy | Still sees and shoots through the shared environment list |
| A9 | Consoles | No errors |

**B, Empty Kit, solo** (three Matches, by `studio-tester`, rendering off):

| # | Check | Result |
|---|---|---|
| B1 | Start | Spawned on `MapSpawn2`. 13 Caches (10 outer, 3 central), 3 Sentries, 0 Pickups. `LootDeal match=1 total=16 counts=ArcBurst:8,TidalBolt:8 variety=ok`. No attribute on any Cache model or descendant names an Ability |
| B2 | An outer Cache with the Basic Attack | 13 swings of 6, all `Hit`, none `killed`. Stage 1 at 45 health, stage 2 at 21, broken at 75. Exactly one Pickup, 0.00 studs across from the marker, the Ability `LootContents` named. The `CacheBroken` line was right. Burst parts, and a `CacheBreak` sound in `PveVisuals`. The stump doesn't collide or take rays, is 1 high, and the hook's `hit` finds `NoTarget`. A second outer Cache held the other Ability, as dealt |
| B3 | Tidal Bolt into a Cache | `Hit` for 12, 75 → 63. The Cache isn't Wet, with no Reaction |
| B4a | A Sentry | The Telegraph attribute appears about 0.47 s before each shot and clears with it. Shots came 3.15, 3.20 and 3.20 s apart. A hit does 12, makes the player Wet, and pushes them |
| B4b | Dodging | A 6-stud sideways move at each of three Telegraphs: no hit from any of them |
| B4c | Breaking sight | Moving 4 studs behind cover cleared the attribute 0.215 s into the Telegraph, before the 0.466 s shot. No `CombatResult` that cycle |
| B4d | Cache cover, the explicit regression | At `CacheCentral1`'s shadow spot, three Sentry shots in a row came back `Blocked` on `cache:CacheCentral1`. The Cache stayed at 120 and stage 0, and the player at 100 |
| B4e | The orb (two-client session, rendering) | It sat on the firing point and grew from 0.46 to 2.20 over 0.49 s, then went with the shot |
| B5 | A central Cache | 20 swings, broken at 120. Tidal Bolt and Arc Burst, 10.00 studs apart, with their midpoint on the marker. From the marker the remaining Pickup was 4.99 away and not collected. A hop toward it collected it. The other had already been collected while swinging from the shadow spot, 3.3 studs from it, as an Upgrade |
| B6 | Killing a Sentry | Tidal Bolt made it Wet, then Arc Burst did 30 with `WetReaction`. It died on Basic Attacks, with `killed=true`, `killRecovery 0` and no health added. `SentryDead`, no longer a target, dark on the client, no Pickup, and `SentryKilled match=3 sentry=sentry:1 by=player:... ability=BasicAttack seconds=77.4`. It didn't Telegraph again in 10 s with the player in its old sight |
| B7 | Eliminated by a Sentry | A real shot: `Eliminated match=2 ... cause=Died by=sentry:3 alive=0 ... dropped=ArcBurst:1,TidalBolt:1`. `PlayerEliminated` has no `killerId`, and the title reads `ELIMINATED`. The drops are marked as from an elimination, at y = 3 and 1.5 studs from the death spot. The `MatchResult` shows `kills=0 reactions=0 dealt=0`, though that Match had a Reaction on a Sentry, about 54 damage to it and 78 to Caches |
| B8 | The next Match | The Pickups were cleared. Then 13 intact Caches, 3 living Sentries, 0 Pickups and a new `LootDeal` (`match=3`) |
| B9 | Consoles | No errors |

**C, two clients** (the main session, rendering at 15 fps):

| # | Check | Result |
|---|---|---|
| C1 | PvE setting up PvP | Sentry1's real bolt hit Player1 for 12 and made them Wet. Player2's real Arc Burst then hit Player1 for 30, with `reactionId=WetReaction`, `attackerId=player:-2`, which is the Reaction's **Triggered By**, and `reactionAppliedBy=sentry:1`, which is the Wet's **Applied By** |
| C2 | A Cache as PvP cover | Player2's Tidal Bolt at Player1, through `CacheOuter01`: `Hit` on `cache:CacheOuter01` for 12 (75 → 63), the Cache not Wet, and Player1 untouched |
| C3 | Results | Sentry1 then eliminated Player1: `Eliminated ... by=sentry:1`, `ELIMINATED`, and `Player2 wins`. Rows: Player2 `Winner`, `kills=0 reactions=1 dealt=30`, only the Reaction on Player1. Player1 `Eliminated`, all zeros |

**Pacing measurements, for Ticket 20** (hops rather than walking, with tool delays). The time from Running to the first Ability was 91.9, 58.2 and 35.3 s across the three Matches, and to a full Kit about 124, 84.0 and 45.4 s. The best case, 35 s and 45 s, is the nearest to real play. The hypothesis above still stands: about 8–10 s to the first Ability with real walking.

**Accepted with notes:**

- **Variety with two Abilities.** The one-swap repair can't reach full variety when both three-Cache chains come out A-B-A: the remaining pair must then match, and fixing it means flipping a chain, several swaps at once. Measured over 2000 seeds, that was 65 partial with two Abilities (3.25%) and 0 with four. Equal counts never broke. `LootRulesSpec` pins 0 for four Abilities and at most 10% for two.
- **`Caches`' lifecycle is `prepare(random)` then `setup(deal, matchId)`,** rather than `setup(random)`. That puts the deal-time check before anything changes in the world, as settled.
- **The Cache attacker gate** uses `MatchRules.canBeHit` for the attacker, at impact, so it's still open during the short end grace, as hits on players are.
- **Sentry knockback** is the tap Tidal Bolt's normal push. In practice it carried players 20–25 studs, often out of the 45-stud range, so a Sentry rarely lands two shots in a row unless the player comes back. It's worth watching when tuning Sentries, and isn't changed here.
- **Unexplained: one `Blocked` Basic Attack** from a test character standing on top of a Cache. A Cache answers `Blocked` only when the gate refuses or it's already broken. Every hit from the ground (B2, B3, B5, C2) behaved normally, so it's left as an edge case to watch rather than a guess.
- **The `CacheBreak` sound** was seen created and playing, but not heard, since every check ran through the MCP.
- **Collection radius** is 3D, from the root to the anchor, so a Pickup at exactly 4.0 across the ground wasn't collected. That's as Ticket 16 settled.

**Published by the developer on 2026-11-09 as version 79**, recorded in the checkpoints above.

### 19. Rift Hook and Stone Guard

- **Scope:**
  - Rift Hook, with forced displacement toward its source and a pull that geometry stops.
  - Stone Guard, with server-authoritative Shield absorption.
  - Rank 2 values and specs for the sequence and burst limits.
  - Both added to the loot bag.
- **Starts only after** the loop works with the existing Reference Abilities, so new content adds to a working loop rather than hiding architectural problems.

**Settled 2026-11-09, before building.** Settled from the docs first, then checked against the code once the Studio MCP was connected. Where the inspection changed a rule's internal shape, the point below is amended and the change is listed under **Reconciled with the code** after it.

1. **Rift Hook's pull.**
   - **Destination:** about 4 studs in front of the caster, on the caster→target line flattened onto XZ and normalised (caster position + direction × 4). Vertical separation plays no part, and the Hook adds no vertical velocity.
   - **Already close:** a target within about 4 studs still takes the hit, the ~8 damage and normal hit interactions, with no displacement. It is never pushed through or past the caster.
   - **Geometry:** a character-sized authoritative sweep from the target toward the destination, against the same static-world roots as projectile line of sight. The pull never passes through or embeds in geometry, and stops short by the target's radius or a safe clearance. Other characters don't block it, and body-blocking isn't part of the Hook this milestone. No collision-safe sweep existed before (a player's Knockback is stopped by client physics, a dummy's by nothing), so the pull adds one small primitive, `Cover:sweep`, a Spherecast against the shared environment roots. It isn't a physics abstraction.
   - **Ledges:** no ledge-stop rule. If the horizontal path is clear, a target may be pulled past supported ground, and gravity takes over. That is horizontal forced displacement, not a vertical launch, and it's deliberate. If Ticket 20 shows environmental Hook kills are too easy or frustrating, revisit map geometry or add a grounded constraint then.
   - **Duration:** a single tuning value, `pullDuration = 0.25 s` (range about 0.2–0.3 s), through the same authoritative forced-movement path as Knockback, which gains an optional duration. Not a teleport, not a slow drag.
   - **Movement Validation** must allow the server-authoritative pull displacement, using the supplied forced-movement duration, rather than treating that movement as invalid player motion.
   - **Caster gone at impact:** if the caster has left, has no usable character or root, or is dead or Eliminated when the Hook lands, the hit and its damage resolve normally, with no pull and no pull Beam. It never pulls toward a corpse, a stale root or the cast position.
   - **Target states:** a Charging target's Charge ends through the existing Knockback rule, with no Hook-specific cancel. An Evading target is unaffected. Frozen and Stunned targets are pulled like any Knockback, if those states exist in the code.
2. **Pulls on non-players.** The pull depends on target capabilities, not target names.

   | Target | Result |
   |---|---|
   | player, dummy | damage + pull, through the server NPC forced-movement path for dummies, generalised rather than a dummy-only Hook |
   | Sentry | damage, no displacement, using the existing "this target can't move" behaviour with no Hook exception |
   | Cache | Hook damage at the caster's Rank through Ticket 18's plain-object path. No pull, Knockback, Status, Reaction, Evade or Kill Recovery |

   - **One target:** the Hook is non-piercing and stops at the first player, dummy, Sentry, Cache or blocking geometry. A Cache or Sentry protects what stands behind it.
   - **No confirmed hit, no pull:** a miss, a wall impact or an Evaded target does no damage and no pull. An Evade doesn't consume the Hook: it flies on and may hit a valid target behind, as every projectile does. Nothing moves on projectile proximity or predicted contact, and the client's cosmetic projectile never moves a target.
   - **Regression invariants:** Sentry immobility must not become Knockback immunity for players or dummies, and the new pull mode must not change Tidal Bolt's push. Same forced-movement foundation, different displacement calculation. Both directions are verified in the code before building.
3. **Stone Guard's Shield.** One server-owned pool, with current Shield, max and expiry. Clients only present it.
   - **Absorption:** each damage instance absorbs from Shield first and takes any overflow from Health, then continues normal death and elimination handling. The Shield is not a separate extra life. A hit that breaks the Shield and overflows can kill.
   - **Reactions:** `Combat` already sums a hit's damage and its Reaction bonus into one damage value, one `CombatResult` and one Session record, so the absorber runs once on that combined value. The Shield totals are identical either way. Reaction events, feedback, `Triggered By` attribution and accounting are unchanged. Shield subtraction lives in one generalised server-side absorber, never inside individual Abilities.
   - **Expiry:** at 3 s or at 0. A timed expiry removes the remainder with no Health conversion, refund or cooldown benefit.
   - **No stacking:** the newest Shield replaces the current one, with no carried remainder.
   - **Evade:** an Evaded hit uses no Shield and counts as no absorbed damage.
   - **Through the Shield:** a valid Shielded hit still applies Wet, Reactions, Knockback and any future Statuses, even at `health damage = 0`. It is never treated as an Evade, and Knockback still interrupts a Charge.
   - **Clearing:** death, Match elimination and a fresh Character lifecycle clear it. Nothing carries into Spectate, the Hub, the next Match or a Reference-mode respawn.
   - **Natural Recovery:** absorbed damage, partial or full, resets the timer.
   - **Tracking:** a fully absorbed valid hit is still a hit, for combat state, killer attribution and fight history. `damageDealt` keeps meaning Health actually lost. A separate `shieldAbsorbed` counter goes in the session log only, not in the Ticket 17 results table.
   - **Low-health feedback** reads Health only.
   - **Replicated state:** character attributes `Shield`, `ShieldMax` and `ShieldEndsAt` (server time, if the client needs it for presentation). Clients never decrement it as authority.
   - **`CombatResult`:** `damage` is Health actually lost, `absorbed` is the Shield consumed by the combined hit and Reaction damage, `shieldLeft` is the Shield after resolution. Examples: 12 into 25 gives 0 / 12 / 13; 20 into 8 gives 12 / 8 / 0; 12 into none gives 12 / 0 / 0. The semantics hold even if the existing result contract needs a slightly different shape, and `Protocol` validation isn't weakened for convenience fields.
   - **Presentation:** a translucent stone-grey shell with warm amber accents that thins with the remaining Shield, readable to opponents, distinct from Evade, Charge, Frozen and Wet, and not hiding the character. A Shield broken by damage gets a break flash and sound. A timed expiry fades quietly. The local player gets a Shield bar beside the Health bar, which doesn't replace it. A fully absorbed hit has its own Shielded hit marker: not a miss, not an Evade, not ordinary damage.
4. **Rank 2.** Both values live in the normal authoritative per-Rank config, resolved by the existing server Kit path. There is no new Rank transport and the client never supplies a Rank.

   | Ability | Rank 1 | Rank 2 | Unchanged |
   |---|---|---|---|
   | Rift Hook | range 30 | range 36 | damage 8, cooldown 7 s, projectile speed and width, 4-stud destination, 0.25 s pull |
   | Stone Guard | Shield 25 | Shield 35 | 3 s duration, 12 s cooldown |

   - **Upgrades affect future Casts only.** An in-flight Hook keeps its parameters. An active Shield keeps its cast-time values, and the next Stone Guard grants the Rank 2 value.
   - **Long pulls:** with the fixed duration, a Rank 2 pull of about 32 studs is faster than Rank 1's. That's accepted for now. If it reads as a teleport in Play, the first lever is a duration that grows with the actual distance, to a small cap. It isn't designed or built now.
   - **No Rank 3 behaviour** for either Ability. If the config schema needs a Rank 3 entry structurally, it follows the existing convention and Milestone 2's cap stays authoritative.
5. **Instant in the Q/E slots.** The behaviour comes from `castingStyle = "Instant"` in the Ability data. Generic Input never branches on `RiftHook` or `StoneGuard`.
   - **Input:** a press Casts once on key-down with the current Aim Point. Release does nothing, holding doesn't repeat, there is no auto-fire, and it never enters Charging. No Tap Window, threshold, slowdown, Telegraph, Charge percentage, release or expiry.
   - **Protocol:** the existing `CastRequest` only. No `ChargeStart` or `ChargeRelease`, no Instant-specific Remote, and strict validation is kept, so extra fields are rejected.
   - **F replacement:** unchanged. While F is held, Q and E are replacement choices and never Cast, for Chargeable and Instant Abilities alike. A key consumed for replacement can't Cast on F's release, since a fresh press is required.
   - **Client suppression** of a slot on cooldown, Equip Locked or Charging is UX only. The server independently enforces every gate, and a rejection wins and spends no cooldown.
   - **While Charging:** an Instant Cast is rejected, spends no cooldown and leaves the Charge running and releasable. It never cancels a Charge, so Stone Guard can't cancel an unsafe Charge into defence and the Hook can't be fired mid-Charge.
   - **Other gates:** the same as other Abilities. Stunned and Frozen refuse every `CastRequest`, the Basic Attack included, as they already refuse `ChargeStart`, Dash and Pickups. Nothing sets either yet, so this changes no live play, and in-flight or committed effects are untouched. In a Match the caster must be Running, Alive, not Stunned or Frozen, past the Equip Lock, off cooldown and not Charging, with the Rank from the authoritative Kit. Reference mode keeps its existing gates and doesn't start requiring a Match.
   - **Aim:** Rift Hook uses the Aim Point and the Ticket 14 hand origin, with its safe-origin and cover rules, and no Hook-specific origin calculation. Stone Guard ignores the Aim Point. `CastRequest` requires one, so the client sends the normal one and the server ignores it, and the Protocol isn't forked. Stone Guard is a new `Self` delivery form, so the `InvalidAim` check is skipped for that form only, never for Projectiles or Sectors.
   - **Prediction:** the Hook may draw a Cosmetic Projectile, cancelled by a Rejected Cast. Collision, damage, target, pull and Charge interruption wait for Server Confirmation. Stone Guard's Shield state is never predicted, and only the cooldown sweep is. A rejection restores the cooldown and shows no Shield.
   - **HUD:** the normal slot presentation (key, name, colour, Rank pips, cooldown sweep, Equip Lock) with no Charge meter and no Shield countdown on the slot. The replacement prompt is unchanged.
   - **No Telegraph** for either. The Hook's counterplay is its visible, narrow, travelling projectile and cover. Stone Guard's is the shell appearing.
6. **Visuals and animation.** Procedural, with built-in placeholder sounds and **no uploads in Ticket 19**: no animation, sound, mesh or VFX asset.
   - **Hook projectile:** a new cosmetic style in the existing Cosmetic Projectile system. It's thin, bright and tether-like, in amber-orange, distinct from Tidal Bolt blue and Arc Burst lightning. No second projectile-rendering system.
   - **Pull feedback:** a rift Beam from the caster's hand toward the target for about the 0.25 s pull, plus a small impact flash. It shows only after the server has confirmed a valid hit, a movable target and an actual pull. A Sentry or Cache hit gets the normal impact feedback and no pull tether. The Beam is cosmetic and never drives movement.
   - **Hook animation:** reuse the uploaded ChargeRelease once on the Cast, with no ChargeHold beforehand. Selection is data-driven, for example a `castAnimation` presentation field in the existing config style, so `CastAnimations` doesn't hard-code the Hook. Chargeable Abilities keep their hold and release behaviour. The code is checked first: if ChargeRelease is tied to Chargeable release, the presentation data is generalised slightly.
   - **Stone Guard:** no character animation, since ChargeRelease would read as offensive. The shell carries the readability. If Play shows the activation feels disconnected, that's a later animation task.
   - **Sounds:** distinct built-in placeholders for the Hook, Guard activation, Guard break and a hit absorbed by the Shield. Each must be unused for another gameplay cue and report `IsLoaded`. Absorbed-hit feedback stays readable without being noisy.
   - **Pickups and HUD:** per-Ability presentation config feeds the Pickup colour and shape, slot colour and name. Hook is amber-orange, Stone Guard is stone grey. Only Cube was free among the existing shapes, so Stone Guard is a `Cube` and the Hook a new primitive `Cylinder`, oriented and spun so it reads as a deliberate Pickup. No mesh.
   - **If the Hook needs its own animation after Play,** key a draft, preview it, bring it back for review, and ask before any upload.
7. **Specs and checks.**
   - **Specs:**
     - **`ShieldRulesSpec`:** the three worked examples (12 into 25, 20 into 8, 12 into none), zero damage, exact depletion, and overflow that kills. A new Shield replaces rather than adds, timed expiry leaves Health alone, and death or elimination clears it. Rank values are 25 and 35, and upgrading mid-Shield leaves the current Shield, `ShieldMax` and expiry unchanged, with the next Cast using the new Rank.
     - **`HookRulesSpec`:** the XZ flattening, so changing only the target's Y leaves the destination unchanged. The destination is about 4 studs from the caster, a close target gets zero displacement and is never pushed past the caster, and no vertical component is introduced. An unobstructed path reaches the full destination, and blocking geometry shortens it by the clearance. Characters aren't supplied as geometry. Range is 30 and 36, and the duration is 0.25 s at both Ranks. If the collision and sweep belong to an existing generalised forced-movement Rules module after inspection, those assertions go there rather than being duplicated.
     - **Combat:**
       - Hit and Reaction damage go through the absorber once, as their combined value.
       - An Evaded hit uses no Shield.
       - A fully absorbed hit counts as a hit, applies Wet and Knockback, triggers a Reaction by the normal rules and resets Natural Recovery.
       - `damageDealt` isn't inflated, and `shieldAbsorbed` accumulates separately.
       - `CombatResult` pins `damage`, `absorbed` and `shieldLeft` with the worked examples.
       - Pull eligibility comes from target capabilities: player and dummy pulled, Sentry damaged but not moved, Cache on its own path. Tidal Bolt's push is unchanged.
     - **`CastsSpec`**, generic through `castingStyle`:
       - An Instant Cast while Charging is rejected, spends no cooldown, and leaves the Charge releasable.
       - Equip Lock, cooldown, Stunned or Frozen, and the Match and Alive gate are enforced server-side.
       - The authoritative Kit Rank is used, a forged `rank` or extra field is dropped, `ChargeStart` for an Instant Ability is refused, and `ChargeRelease` can't manufacture an Instant Cast.
       - Tidal Bolt still pushes away while the Hook pulls toward its safe destination.
     - **`LootRulesSpec`:** the real pool is four Abilities with four copies each (16 drops). The seed sweep, 200 seeds routinely and about 2000 if cheap, otherwise as a separate stress check, requires for every seed exact 4/4/4/4 counts, central Cache entries that differ, every drop assigned once, a repair that terminates, the same seed reproducing the same deal, and a reported `variety=ok|partial` that matches the nearest-pair state. **Zero partial cases isn't a contract** unless the algorithm and geometry guarantee it. Equality stays strict and early variety stays best effort, so a zero from the sweep is evidence only.
     - **Burst limits:** the values are calculated from the authoritative Ability config, with no handwritten totals pinned. Principles: no ordinary single hit above about 40% of 100 Health, and no intended setup and payoff above about 60%, for every Hook and Guard pairing at the configured Rank values. The Hook is untyped and creates no Wet or Reaction setup. Stone Guard only adds defence and can't raise offensive burst.
   - **Play checks, by `studio-tester`, with the render and state inspected before each interaction group:**
     - **Hook:**
       - A normal hit pulls a player to about 4 studs in front of the caster.
       - A target already inside that distance takes the hit and doesn't move.
       - A wall shortens the pull safely, without embedding the target, and characters don't body-block.
       - A pull past a ledge is allowed, with no artificial vertical launch and gravity taking over.
       - A miss, a wall impact and an Evaded target give no damage and no pull Beam.
       - A Charging target is hit and pulled, and its Charge ends through generalised Knockback with no Hook shortcut.
       - A Sentry takes about 8 and doesn't move, a Cache takes Hook damage with no Status, Reaction or displacement, and a dummy is pulled through the server NPC path.
       - Rank 2 range, with a target around 33 studs and near the boundary: Rank 1 can't hit and Rank 2 can.
       - Movement Validation shows no violation or snap-back after short and long pulls, with the long Rank 2 case inspected.
     - **Stone Guard:**
       - After a Cast, the replicated attributes and the shell are correct.
       - Tidal Bolt into Guard absorbs first, still applies Wet and Knockback, shows as a real Shielded hit and resets Natural Recovery.
       - A real Wet → Arc Burst sequence is tested where the hit takes some Shield, the Reaction takes the rest and its overflow reaches Health, with attribution unchanged.
       - A timed expiry with some Shield left fades quietly with no Health change.
       - A break shows the flash and sound and clears the attributes.
       - Replacement is tested through the smallest legitimate setup, without changing production cooldowns, since the pure spec is the main contract.
       - Death clears the Shield, and an Evaded hit consumes none.
       - A mid-Shield upgrade leaves 25 / 25 unchanged, and the next Cast gives 35 / 35.
     - **Instant input**, both Abilities on real Q and E:
       - One key-down gives one Cast, holding doesn't repeat, and release does nothing.
       - While F is held, Q and E only replace.
       - Releasing F with the key still held doesn't Cast, and a fresh press is required.
       - A press during a Charge is refused with no cooldown spent and the Charge continuing.
       - Forged and malformed requests go through the existing Protocol test path.
     - **Loot and upgrades:**
       - One Match deal holds Tidal Bolt, Arc Burst, Rift Hook and Stone Guard four times each, taken from `LootDeal` or `LootContents`, and one player needn't collect all 16.
       - Enough Caches are broken to see both new Pickups, with correct names, distinct colours and shapes, and replacement prompts naming them.
       - A duplicate upgrades an owned Rank 1 Ability to Rank 2, and the new values apply to the next Cast.
     - **Reference Kit regression:** Tidal Bolt pushes as before, Chargeable input and Arc Burst are unchanged, and the dummies behave as before except where the Hook is deliberately granted. No Match-only system disturbs Stage 3.
   - **Main-session review after the runs** (judgement calls, not delegated):
     - **Long Rank 2 pull:** fast forced movement (keep) or teleport-like (record for tuning). The duration isn't changed in this ticket unless clearly broken.
     - **Shield shell:** visible to opponents, distinct, the silhouette stays readable, and the draining is understandable.
     - **Guard audio:** activation, absorbed hit and break are distinguishable and repeated hits aren't noisy.
     - **Hook projectile:** visible against the Reference arena, `MatchMap` and common cover. No asset is uploaded because a placeholder needs tuning.
   - **Ticket 20 watch list** (recorded, not solved):
     - **Stone Guard effective health:** +25 or +35 temporary capacity changes the Tidal → Arc payoff and overall time to kill. Guard isn't weakened pre-emptively, because absorbing much of a combo is its defensive identity.
     - **Environmental Hook kills:** ledge pulls, falls caused by the Hook, and whether they read as fair and how often they happen. No ledge protection in this ticket.
   - **Test-aid hygiene:**
     - Temporary helpers may grant Abilities or Ranks, set Health, arrange Shield scenarios or position players.
     - The mechanic under test uses the normal authoritative paths.
     - Before finishing, the helper is deleted and a search of the runtime code for its exact script and API names finds nothing. The word `TEMPORARY` isn't searched repo-wide, because these docs legitimately contain it.
     - Any temporary tuning is reverted, `startingKit` is back to `"Reference"`, and the aid is never committed.
   - **Done when:**
     - The specs pass: `ShieldRules`, `HookRules`, the updated system specs, four-Ability loot and the burst limits.
     - The Hook, Guard, Instant Q/E and Reference regression checks pass, and Movement Validation accepts the pulls.
     - The Hook and Shield readability has been reviewed.
     - The temporary aids are gone, `startingKit` is `"Reference"`, and `code-structure.md` and `studio-mcp-quirks.md` carry anything new.
     - No unapproved asset was uploaded, and nothing from Ticket 20 has started.

**Reconciled with the code 2026-11-09, after inspecting it and before building.** The developer approved each change.

- **Forced movement gains an optional duration.** `CombatResult` gets an optional `knockbackSeconds`. The client's Knockback, Movement Validation's allowance and the dummies' forced movement use it when present and `Tuning.knockback.durationSeconds` (0.2 s) when absent, so Tidal Bolt's push stays on its existing path. Knockback isn't changed to 0.25 globally.
- **The pull's destination.** Pure `HookRules` owns the flattened direction, the destination 4 studs in front of the caster, the zero pull when close, the clearance, the 0.25 s duration and the ranges. `Cover:sweep(from, to, radius)` is injected, ignores characters, and returns a safe end short of static geometry. The Hook's Knockback is `velocity = safeDisplacement / 0.25` with `knockbackSeconds = 0.25`. A dummy gets the already-clipped velocity and never raycasts by itself.
- **Caster gone at impact:** damage only, no pull and no Beam.
- **One absorber call** on the combined hit and Reaction damage. The order is Evade, the Reaction, the total damage, the Shield, Health, `noteDamage`, then Statuses and Knockback as before.
- **A `Self` delivery form** for Stone Guard: on the server it grants or replaces the Shield at the Kit's Rank through the normal cooldown, with no target, projectile or Sector. On the client it draws no delivery geometry. The `InvalidAim` check is skipped for `Self` only.
- **A generic Stunned and Frozen gate** on `CastRequest`, the Basic Attack included, with a Reference regression check.
- **Pickup shapes:** Tidal Bolt Sphere, Arc Burst Diamond, Stone Guard Cube, Rift Hook Cylinder.
- **An Evade lets the Hook fly on,** like every projectile.
- **Loot variety stays best effort.** `LootRulesSpec` drops its requirement of zero partial deals for four Abilities.
- **Small presentation extension points:** `visual.castAnimation` chooses a Cast's animation instead of `Casting` checking the Casting Style and the Basic Attack's id, and `visual.projectileStyle` chooses a Cosmetic Projectile's look.
- **The Shield is cleared on death** explicitly, as well as by a new character, because an Eliminated body stays for 3 s.

**Built and tested 2026-11-09. All checks pass.**

- **Server side, built by the main session:**
  - **New pure Rules:** `ShieldRules` (absorption, a new Shield, whether it's active, and whether a hit broke it) and `HookRules` (the destination, the clipped displacement and the velocity).
  - **New system:** `Shields`, the one owner of each character's Shield and its attributes, with `grant`, `absorb`, `clear`, `get` and `deliver` (the `Self` form's delivery).
  - **`Cover:sweep(from, to, radius)`:** a Spherecast against the environment roots that returns the farthest centre position.
  - **`Combat`:** one `shields.absorb` call on the combined damage, and the `Pull` branch working from the caster's current root. `CombatResult` gained `absorbed`, `shieldLeft` and `knockbackSeconds`.
  - **Forced movement:** `Movement.applyKnockback` and the dummies take the duration. `Characters` gained `immovable`, set for Sentries, and clears the Shield on `Died`.
  - **`Casts`:** the `Self` form, the generic Stunned and Frozen gate, and a `manual` start with `Casts.receive` for specs.
  - **Session:** a log-only `shieldAbsorbed`.
  - **Config:** `RiftHook` and `StoneGuard`, `Tuning.hook`, `Tuning.shield.display`, four placeholder sounds, the four-Ability loot pool and the `Cylinder` Pickup shape.
- **Client side:**
  - **Main session:** `Knockback` takes the duration, and `Casting` the `Self` form and `visual.castAnimation`.
  - **`studio-implementer`, from an exact brief, reviewed before acceptance:**
    - the new `ShieldVisuals` (shell, amber ring, thinning, break and quiet fade)
    - `Hud`'s Shield bar, read by `HealthDisplay`
    - `Feedback`'s Shielded hit marker, the absorbed-hit sounds and the pull Beam
    - the `Tether` Cosmetic Projectile
    - the Cylinder Pickup
- **Fixes during review and testing:**
  - **Break or fade.** The server clears the Shield attributes before it fires the hit's `CombatResult`, so the attribute can arrive first. `ShieldVisuals` waits 0.15 s before choosing a break or a quiet fade.
  - **The player's pull overshot.** Time alone ended it a frame late: 0.8 studs, and once 2 studs into body contact with the caster. A movement with its own duration now also stops on arrival on the client, slowing its last frame. Tidal Bolt's push is still timed only.
  - **The dummy's pull fell short.** A long server frame cut it to 5.1 studs. A dummy's pull now travels exactly its planned distance, and the measured end is 4.00.
  - **Colours.** Neon lightened the amber accent ring and the Hook's head to yellow, too close to Arc Burst's. Both now use a deeper amber (215, 110, 30), the Hook's Pickup and slot colour too, and the tether line is 0.15 wide rather than 0.12.
- **Specs: 394 passed, 0 failed,** on the Server in Play, solo and in the two-client session.
  - **New:**
    - `ShieldRulesSpec` (5)
    - `ShieldsSpec` (6)
    - `HookRulesSpec` (7)
    - `CastsSpec` (7)
    - `BurstLimitsSpec` (3)
  - **`CombatSpec`:** 9 new tests for the Shield worked examples, the fully absorbed hit, the combined Reaction, Evade and overflow, and the pull on players, dummies, clipped dummies, Sentries and Caches, with the caster gone or the target close, and Tidal Bolt against the Hook.
  - **`CastRulesSpec`:** the `Self` form and the cast animations.
  - **`CachesSpec`:** the four-Ability `LootDeal`.
  - **`LootRulesSpec`:** 2000 seeds (0.09 s), with no requirement of zero partial deals. It measured 65 partial with two Abilities and 0 with four.
- **Burst limits,** from config:
  - the worst single hit is Arc Burst Rank 2, at full Charge, on a Wet target: 18.4 + 17.6 = 36
  - the worst two-slot pair is Tidal Bolt then Arc Burst at Rank 2: 52.25
  - with the Hook, the worst is 26.4
- **Test setup:**
  - **The aid:** a TEMPORARY `TEMPORARYHookAid` server script, with a BindableFunction, offered `cast` through `Casts.castFromServer` (a real projectile from a dummy), `setSlot` and `grant` through `Kits`, `setHealth` and `spawnPickup` through `Pickups.spawn`.
  - **The Starting Kit** was `Empty` for Run B only.
  - **Afterwards:** the aid is deleted, no instance or script source names `HookAid`, and `startingKit` is `"Reference"`.

**A, solo, by `studio-tester`** (Reference Kit, rendering at 15 fps):

| # | Check | Result |
|---|---|---|
| A1–A3 | Reference regression and the new gate | Tidal Bolt 12 with its normal push and no `knockbackSeconds`. Arc Burst 30 with `WetReaction`. The Basic Attack hit for 6. `Stunned` and `Frozen` each refused a Basic Attack, which worked again once cleared |
| H1, H2 | The Hook on a dummy | 8 and a 0.25 s pull toward the caster (5.1 studs before the dummy fix, then 4.00 after it). Within 3 studs: 8 and no move |
| H3 | Rank 2 range | At 34.96 studs, Rank 1 missed and Rank 2 hit and pulled to 3.96 |
| H4 | The player pulled | 24 → 3.80 studs in about 0.27 s, with no later snap and no Movement Validation correction |
| H5 | A Charging target | Pulled, with `ChargeEnded ... cause=Knockback` |
| H6, H7 | A miss, and an Evade | No damage, no move and no Beam. `Evaded` with no pull |
| G1, G2 | Guard, then a Tidal Bolt into it | 25 / 25 for 3.0 s, with the shell, ring and bar. Then 0 / 12 / 13: Health 100, Wet and a push |
| G3 | The combined Reaction | `WetReaction`, Triggered By and Applied By both `dummy:StaticDummy`, absorbed 13, `shieldLeft` 0, `damage` 17, Health 83 |
| G4, G5 | Break and expiry | A break grew and faded the shell over 0.35 s with the `glassbreak` sound. An expiry faded quietly with Health unchanged |
| G6 | Natural Recovery | An absorbed hit held Health at 91.9 for 7.1 s before recovery |
| G7, G8 | Evade and death | An Evaded hit left the Shield at 25. Death cleared it in 0.05 s, with none after respawn |
| G9, G10 | Upgrade and replacement | A mid-Shield upgrade kept 25 / 25, and the next Cast gave 35 / 35. A replacement drew `EquipLocked` (0.75 s left), then Cast |
| I1, I2 | Instant gates | A Hook during a Charge was refused as `Charging`, and the next one wasn't on cooldown. `ChargeStart` for the Hook gave `NotChargeable`, and an extra `rank` field was dropped with a warning |
| — | Session | The `Summary` had `shieldAbsorbed: 49`, which matches the absorbed hits |

**B, Empty Kit, solo, by `studio-tester`:**

| # | Check | Result |
|---|---|---|
| B1 | Deal | `counts=ArcBurst:4,RiftHook:4,StoneGuard:4,TidalBolt:4 variety=ok` |
| B2 | Pickups | Hook: an upright amber Cylinder. Guard: a grey Cube |
| B3, B4 | Sentry and Cache | A Sentry took 8 with no knockback and no move. A Cache 75 → 67 with no Status |
| B5 | Upgrade | A second Hook upgraded to Rank 2 and hit at 34.6 studs |

**C, two clients, by `studio-tester`** (Server & Clients, rendering at 15 fps):

| # | Check | Result |
|---|---|---|
| K2–K4 | Real Q/E input | One press, one Cast. A 2 s hold gave one Cast and the release none. E gave a Shield of 25 |
| K5, K6 | The F modifier | F held: no Cast. F released with Q still held: no Cast, and a fresh press Cast once |
| K7 | E during a Tidal Bolt Charge | Nothing, while the Charge released. E right after Cast normally |
| P1h | Short pull, player on player | 17.56 → 3.80 studs in 0.263 s, with no correction |
| P2h | Long Rank 2 pull | About 30 studs, 33 → 3.75, in 0.264 s at about 2 studs per frame, with no correction |
| P3h | A wall | Pulled from the low area toward a caster above, it stopped at 1.84 from the ledge face (the 1.5 radius plus the 0.5 clearance, as clipped), with y unchanged |
| P4h | A ledge | Pulled off the lip, it fell from y 3.2 to -4.8 with no upward launch and no correction |
| P5h | A dummy near the path | The dummy didn't stop the pull, which ended about 3.9 from the caster |
| S1–S3 | The other client | The shell and ring showed. The Tether head and Beam showed and cleared at range end. `PullHand` lasted 0.25 s |

**Main-session review (two-client screenshots):**

- **The long Rank 2 pull:** fast forced movement, not a teleport, about 16 frames over 30 studs. Kept, with the duration unchanged.
- **The Shield shell:** a translucent grey bubble that keeps the character, the name and the silhouette readable. It's distinct from Evade, Wet and the Charge orb, and the thinning reads. Its accent was changed from yellow to deep amber, as above.
- **The Hook projectile:** a thin line from the hand with a bright elongated head, visible against the arena's floor and cover. Its colour was changed as above. Not reviewed on `MatchMap`, but it uses the same environment colours.
- **The Guard audio** (`metal.ogg` on, `hit.wav` absorbed, `glassbreak.wav` break) was chosen distinct and checked to load and play, but not heard, since every check ran through the MCP. It's for the developer to listen to.

**Accepted with notes:**

- **A close target isn't Knockbacked,** so a Hook within 4 studs doesn't end a Charge. That follows "no displacement", and nothing else would push it.
- **The player's own pull ends on arrival on their client,** so where they stop still depends on how current the server's view of both players was at impact. Measured ends were 3.75–3.96 studs.
- **Characters aren't in the sweep,** but a player's own physics still collides with a body directly on the pull line. Not tested exactly on the line.
- **The `Shield` attributes clear on the server before `CombatResult` fires.** Clients allow for it, and it's recorded in `code-structure.md`.
- **Ticket 20 watch list,** carried from the settlement: Stone Guard's effective health against the Tidal → Arc payoff, and environmental Hook kills off ledges.

**Published by the developer on 2026-11-09 as version 86**, recorded in the checkpoints above.

### 20. Run test readiness

- **Scope:**
  - Stage 1 and 2 regression on the whole progression loop, with forged and flooded `ReplaceRequest`s and the other authority and security checks.
  - A Reference Kit regression.
  - Pacing logs: time to first Ability and to a full Kit, Upgrades, replacements, central encounters, eliminations and timeouts.
- **Publishing:** this prepares a publish candidate and stops for the developer's approval. Nothing is published as part of finishing the ticket.
- **The sitting with outside testers** still waits for Stage 3 to validate the combat.

**Settled 2026-11-09, before building.** Settled with the developer from the docs, then checked against the code where noted. Nothing is built yet.

1. **What "Stage 1 and 2 regression" means here.** It does **not** mean rerunning every acceptance check from Tickets 15–19. The 394 specs and the earlier tickets' Play checks stay as evidence, and Ticket 20 adds integration confidence without duplicating them. It means two runs on the final Ticket 19 code:
   - **Run R, the Reference Kit regression.** It protects Stage 3 and the shared combat paths changed since Ticket 14. Solo first, then two clients where real replication matters.
     - **Covers only shared paths:** Basic Attack, Dash, Evade, Tidal Bolt, Arc Burst, the Wet → Arc Burst Reaction, Charging and its interruption by Knockback or forced movement, `CastRejected` and Dash reconciliation, and the player-vs-player replication those need.
     - **Confirms the progression systems are dormant:** no Match lifecycle, no Hub participation state, no runtime Caches or Sentries, no `LootDeal`, and the arena and dummies remain the Reference environment.
     - **Not judged through the MCP:** aim feel, projectile feel, readability or fun. Those stay with the developer and Stage 3.
   - **Run M, the final progression-loop integration**, with two clients and an Empty Kit: Hub → Countdown → `MatchMap` → PvE and loot → Kit building → PvP → Elimination → Results → Hub. The goal is that the individually tested systems work together, with natural play preferred to isolated setup.
     - **Match 1, the Winner path:** both players start in the Hub, the countdown starts the Match, spawns are distinct, Caches and Sentries initialise, and there are no loose Pickups. Then Caches are broken, Abilities collected, a duplicate Upgrades, a replacement is made and a Sentry is met. At least one Ticket 19 Ability is used in the PvP fight, which ends in an elimination and the whole-Kit drop. The other player wins, the Results rows and logs are right, both return to the Hub and the Match is cleaned up. Not every Ability pairing is needed.
     - **Match 2, the Timeout path:** `maxSeconds` is shortened temporarily, both players are Alive at the end, and the result is Timeout with no winner invented and both survivors shown. Results and the Hub cleanup still work, and `maxSeconds` goes back to 480 straight away.
     - **Leaving mid-Match,** as a third short Match or folded in, whichever is cleaner, without weakening the Winner and Timeout checks. It shows `Left`, the whole-Kit drop before cleanup, no double elimination, the remaining player's win, and the leaver's result row surviving the departure.
   - **Version discipline.** Both runs use the same candidate code where possible. A fix to shared gameplay after Run R reruns the affected part of Run R. A fix to progression-only code reruns the affected part of Run M. The whole matrix isn't restarted after every local fix.
2. **Pacing and measurement logs.** All use the one authoritative Match clock, **seconds since `Running` began**, the same as `CacheBroken`, `SentryKilled`, `Eliminated` and `MatchEnd` already do. Never client time. Nothing new reaches the world, the clients or the attributes, and Reference mode logs nothing new. Checked in `Session`: `Pickup` had no timing, and `MatchResult` already had per-player `equips`, `upgrades` and `replaces`.
   - **`Pickup`** gains `seconds=%.1f filled=%d/%d`, appended after the existing fields. `filled` is the number of non-empty Ability slots after the mutation over the configured slot count. An Upgrade or a replacement of a full Kit stays `2/2`, and a Rank never fills a slot.
   - **`MatchResult`** gains `firstAbility=<seconds|none>` and `fullKit=<seconds|none>`, plus `guards=%d` and `shieldAbsorbed=%.0f` (item 4).
     - **`firstAbility`** is set once, at the first move from 0 to at least 1 filled slot, judged from the Kit's actual occupancy and not from the Pickup's kind.
     - **`fullKit`** is set once, at the first move to every slot filled.
     - **Neither changes later,** whatever happens to the Kit: a replacement, an Upgrade or the elimination drop. They are history and not state, and they survive elimination, `ReturnToHub` and the leaver's row captured before Session cleanup.
   - **Ownership.** The milestones are stored with the player's Match data in `Session`/`Match`, recorded by `Session.recordPickup`, and serialised once when the result row is built. They are never parsed back out of log strings.
   - **Central fights.** The existing fight definition is reused, with no parallel encounter state.
     - A fight is `central=yes` when the **first damaging player-vs-player hit's target position** is within `Tuning.session.centralRadius` of the `MatchMap` centre, flattened to XZ. The value is **50**, with `distance <= centralRadius` as the boundary. For comparison, the central Caches sit at about r 26 and the Sentries at 20, the outer Caches at 80 and the spawns at 105.
     - The tag is stored once on the fight record and never reclassified. A fight that starts outside and moves in stays `no`. One that starts in the centre and moves out stays `yes`. A fight that times out after 7 s and then restarts is a new fight with a fresh tag.
     - Sentry or Cache damage alone never starts a PvP fight. A hit from a player on someone a Sentry has made Wet does.
     - Inside a Match, `Fight` lines gain `match=N at=%.1f central=yes|no`. Outside a Match the format is unchanged, with no `match=none` or placeholder fields.
     - `MatchEnd` gains `fights=%d centralFights=%d`, counted once per fight. A lone hit that nothing followed isn't a fight under the existing rule, so a fight counts when it is recognised, and `MatchEnd` closes any open fight first so the totals are complete.
     - **No `cacheStartedFight` field.** The relation is read afterwards from `CacheBroken kind=Central seconds=X` against `Fight at=Y central=yes`. `kind=Outer|Central` already exists on `CacheBroken`, and the spec checks it is reliable.
3. **The authority and security checks.** Run on a second client's real remotes by `studio-tester`. Combat requests were forged in earlier tickets, so this covers the progression-specific paths and the gates that only matter in a Match. Per remote tested, the record says: guarded yes or no, the guard parameters, the malformed outcome, the flood outcome, state integrity, recovery and any finding.
   - **`ReplaceRequest`**, the one handler confirmed to have a guard (a token bucket, `burst = 5`, `perSecond = 5`, dropping excess before validation).
     - **Flood:** 50 in one tick, and then 50 spread over several seconds. In one tick at most 5 reach validation, with no duplicated mutation. Sustained throughput is bounded approximately by the refill rate, allowing for scheduler variance. A legitimate request works promptly once tokens are available, and a flood doesn't starve it for an unreasonable time after the burst ends.
     - **Malformed:** a string where an integer is expected, a non-integer, NaN, infinity where the validator can test it, a negative, slot `0` and `3`, an unknown `pickupId`, an extra `abilityId` or `rank`, and a missing field. Where the Protocol layer rejects first, "dropped as malformed" is the expected result, with no reason required. A structurally valid but semantically bad request goes through the handler's own refusal. No single rejection mechanism is forced.
     - **State gates:** more than 6 studs away, a Kit that isn't full, Charging, Eliminated, in the Hub, a voluntary Hub spectator (refused because the server's Match state is `Hub`, not because of what the client says), and no Running Match. A refusal drops no outgoing Ability, leaves the Pickup, and leaves the Kit, Equip Lock and cooldowns unchanged.
     - **Race:** two clients take the same Pickup in the same server step, as far as the harness allows. Exactly one replacement succeeds, the Pickup is removed once, there is one outgoing drop, the loser sees no mutation, and nothing is duplicated. The winner's identity isn't pinned.
   - **`ReturnToHub`** has no flood guard, because Ticket 17 didn't require one, and none is added now. An Alive player in a Running Match is never removed by it. A Hub player stays in the Hub, a payload with extra fields is dropped, and repeated calls after a legitimate success are harmless, with no duplicate spawns or state corruption, reasonably throttled warnings and no material degradation.
   - **Gates outside active play:** `CastRequest`, `ChargeStart` and a Basic Attack from Eliminated, Hub, Hub-spectator and no-Match states, and an Ability that isn't in the authoritative Kit. All give no combat effect, no cooldown spent, no Charge, no Health or Shield change and no target damage. **`DashRequest` follows the settled Ticket 17 gate,** which allows movement-only Dash in the Hub and during Results, so it is not required to be rejected there. Eliminated or invalid character state must not cause an invalid mutation.
   - **Cache damage** from a Hub player, an Eliminated player and outside Running, through the real attack path where practical. Health and `CacheStage` stay unchanged, no Pickup appears and no `CacheBroken` is logged.
   - **`CastRequest`, `DashRequest` and `ChargeStart` floods are measured only,** at about 50 in one tick. First check whether each already has a generic guard, and don't assume there is none. If one does, record its parameters and observed behaviour. If one doesn't, **no guard is added in this ticket.** Measure server errors, Heartbeat responsiveness, tick degradation, state corruption, log volume and whether a legitimate request still works straight afterwards.
   - **Pass, all five:**
     1. Authority: every forged request is rejected, ignored or dropped at the right layer, with a reason verified only where one exists.
     2. No unauthorised mutation of the Kit, Pickups, Rank, Equip Lock, cooldown, Match phase, roster, Health, Shield or Cache/Sentry state. Movement is allowed only where the design allows it.
     3. Race safety: one winner and no duplication.
     4. Stability: no unhandled exception, runaway task, pathological warning spam or permanently broken state. Throttled warnings from deliberate malformed tests are fine.
     5. Recovery: Heartbeat stays responsive, and a legitimate request works once the normal cooldown, token refill and Match state allow it. A guarded request doesn't have to bypass an exhausted bucket.
   - **A Ticket 20 finding,** classified and brought back before any generic rate-limit framework is considered, is an unguarded flood that causes sustained Heartbeat degradation, Match-loop starvation, state corruption, excessive unthrottled logs, requests still unusable after the burst, or a server error. A brief spike that recovers immediately isn't a finding.
4. **The watch list is measured only, with no automatic tuning thresholds.** A handful of testers is too small for pass or fail, and the MCP can't judge fairness. Ticket 20 makes the effects observable, and the run test decides.
   - **Stone Guard.** `MatchResult` gains `guards` (successful Stone Guard activations only, never rejected, malformed or Equip-Locked attempts) and `shieldAbsorbed` (Shield absorbed in that Match, as a Match-local delta like the other session stats). `damageDealt` keeps meaning Health damage dealt, and Results UI doesn't change.
     - **Static context,** recorded as analysis and not as an invariant (`BurstLimitsSpec` already pins the Ability values): the worst two-slot burst, 52.25, is 39% of 135 effective health with a Rank 2 Guard, against 52% of 100 without one.
   - **Environmental Hook kills.**
     - **`HookPull match=N caster=… target=… at=… studs=…`:** one line per Hook that actually displaces a player. `studs` is the authoritative safe displacement, not a client estimate. It is not logged for a miss, a wall impact, an Evade, a Cache or Sentry hit, a zero-displacement close target, or a damage-only hit with the caster gone.
     - **`hookPulledAgo` and `hookBy` on `Eliminated`:** `Session` keeps the last successful player pull per target (the target, the caster, the Match and the time), replaced by each newer pull. When a player is eliminated with no valid player killer and was pulled within **5.0 s**, the line carries `hookPulledAgo=<seconds> hookBy=<player>`, otherwise `hookPulledAgo=none`.
     - **Telemetry only.** It is correlation and not kill credit. It never becomes the killer, a PvP kill, `PlayerEliminated.killerId` or a Results `eliminatedBy`, and the existing attribution rules are unchanged. The record clears on death, elimination, a fresh Character, Return to Hub and a new Match, so Match N never annotates Match N+1.
   - **Defects versus observations.** Fixed in Ticket 20 if seen: a target embedded in a wall, a pull through blocking geometry, Movement Validation snapping a valid pull back, an invalid vertical launch, or a duplicated or wrong displacement. Recorded for the run-test review without tuning: how often Hook leads to falls, whether ledge pulls feel fair, Guard's durability and fight outcomes, and Shield efficiency.
   - **Review questions for the run test, not triggers.** How many Guard casts and how much Shield absorbed per Match? Do Guard users dominate otherwise even fights? How many Hook pulls, and did any no-killer elimination come within 5 s of one? Did testers call those deaths readable and fair?
5. **How pacing is measured.** There are two separate questions: the **mechanical floor** (how fast can a player who knows the route get the first Ability and a full Kit) and **human run pace** (orienting, fighting Caches, Sentries, PvP, loot choices). Scripted walkers measure only the first, and are never presented as expected human pacing. The authoritative Session logs are the timing source, and the script's own timing is never a competing one.
   - **Scripted real-speed walkers,** armed in an earlier call on each client before Running and waiting on `MatchPhase == Running`. Each uses `Humanoid:MoveTo` through the real `MatchMap` and any waypoints around real cover, with no teleporting, so it stays subject to collisions, Movement Validation and movement-state limits. At each Cache it approaches into Basic Attack range, sends real Basic Attack `CastRequest`s at the normal cadence, breaks it, lets the Pickup system collect the drop, and walks on to its second suitable outer Cache. The test hook never damages Caches for this measurement.
   - **Three Matches, both clients at once,** from different spawn pads, for a spread. Record per player the pad, `firstAbility` and `fullKit`, with notes.
   - **One central pass,** with the Sentries left active, since their pressure is part of the central cost. Record separately the travel time to the centre, the Cache break timing, and whether Sentry pressure interrupted or slowed it. If the player dies or PvP disrupts it, the pass is rerun and the disrupted value isn't taken as the baseline.
   - **Movement Validation check:** the first run verifies that real-speed `MoveTo` produces no violation, no snap-back and no abnormal speed correction.
   - **If `MoveTo` conflicts with the harness,** do not fall back to hops. In order: confirm the failure really is Movement Validation and not path geometry, adjust the walker to drive normal Humanoid movement, use a small helper that still moves continuously at the real WalkSpeed, and only if that can't be made reliable use a **synthetic floor** (actual path distance ÷ configured speed, plus real Cache and action timing), labelled as such and never as a Play measurement.
   - **Reporting.** A compact table (Match, player and spawn, first Ability, full Kit, notes), then the minimum, median and maximum of the outer runs, and the central pass separately. Values are labelled **floor pacing under perfect route knowledge,** excluding search, hesitation, reading Pickups, loot decisions, avoiding players, aim and detours. They are compared against **both** references and neither replaces the other: Ticket 18's hypothesis (first Ability about 8–10 s, full Kit about 25 s, from geometry and Cache break time) and Milestone 2's design targets (first Ability about 30–45 s, full Kit about 1.5–2 minutes). The useful question is how far apart the floor and the intended human experience are. There is no pass or fail from three Matches.
   - **No config change without review.** Ticket 20 measures and reports first, and brings back the ranges, the central cost and a recommendation. It changes none of Cache Health, Cache positions, Cache count, spawn positions, movement speed or the collection radius without the developer's approval. It also names where the time actually goes, and doesn't assume Health is the lever. A scripted floor that misses the human target isn't a reason to tune, since the human run test is decisive.
6. **Publish candidate and the run-test switch.** A Ticket 20 **Reference-safe publish candidate** is the exact Studio state at a recorded git commit where:
   - `RunAll` passes on the Server in Play, with the telemetry specs in the count
   - Run R and Run M have passed on the same code
   - no temporary runtime aid remains, with the exact script and API names searched and absent from the runtime and source paths
   - all temporary tuning is restored, `Tuning.match.maxSeconds = 480` and `Tuning.kit.startingKit = "Reference"`
   - the git tree is clean
   - the ticket records the commit, the final spec count, the Run R and Run M results, the telemetry findings and what changed since version 86

   It is a tested state and not a Roblox version, and it is never left in Empty mode. The repo's `.rbxl` stays the Ticket 1 baseline, and Roblox version history stays the checkpoint record. The Checkpoints row is added only once the developer has published and reported the version, and it records both the git commit and the Roblox version. A version is never inferred from git.

   **Run test switch.** There is no runtime selector, place attribute, second flag or mode framework.
   - **To prepare the progression run test:** change exactly `Tuning.kit.startingKit = "Reference"` to `"Empty"`, check that no other code or config change exists, publish manually, and record the resulting version in Checkpoints.
   - **After the run test, before any Stage 3 sitting:** restore `"Reference"`, check it is the only intended mode change, publish again and record that Reference-safe version.
   - **Never assume the currently published version has the right `startingKit`. Check it before each external sitting.**
   - Run M may set `"Empty"` in Edit mode for testing and reverts it afterwards. Ticket 20 doesn't publish an Empty build to prove it works.

   **Pre-publish checklists,** because one published place serves both modes:
   - **Reference / Stage 3:** `startingKit = "Reference"`, `maxSeconds = 480`, no temporary hooks, a clean tree at the expected commit, and `RunAll` passed.
   - **Progression run test:** the same candidate code, the only mode change `startingKit = "Empty"`, no temporary hooks or tuning, and the resulting version recorded.

   **The final handoff** states the commit, the spec count, the candidate's `startingKit`, the Run R and Run M status, any unresolved findings, and that publishing is manual.
7. **Ticket 19's leftovers.** Closed in this order: the sound listening pass, any sound swap, Run R, then Run M with the Hook and body checks inside it. No extra Match is needed for them.
   - **Placeholder sounds,** the one leftover that needs human judgement. Before Run M, the developer listens through one short solo Play sequence: Cast Rift Hook, Cast Stone Guard, take a hit while Shielded, let damage break the Shield, then Cast Guard again and let it expire.
     - **Each of the four cues** (Hook cast, Guard activation, absorbed hit, break) gets *keep* or *replace*, judged on audibility, volume, distinctness from nearby cues, repeated absorbed hits not annoying, the break clearly stronger than an absorbed hit, and Guard activation not sounding offensive. A timed expiry stays quiet and doesn't use the break cue.
     - **A replacement is another built-in `rbxasset://` sound** that isn't used for another important cue and reports `IsLoaded`, and is listened to again. No uploads, and no production audio pass.
     - **Closed when** all four have a verdict, any rejected cue is replaced and re-heard, all retained sounds load, and nothing was uploaded.
   - **Hook on `MatchMap`.** `studio-tester` captures representative screenshots of real Hook use in Run M against the open floor, the middle-ring cover and the central core, at normal combat distance. The main session asks only whether the projectile and head can be tracked, whether the pull line is visible after confirmation, and whether it stays distinct from Tidal Bolt and Arc Burst. A small procedural tweak (colour or value, brightness, width, head or trail scale) is allowed only if it is clearly lost against the environment, and is rechecked. Anything subtler goes into the run-test notes, and the effect isn't redesigned.
   - **A body on the pull line.** In Run M's two-client checks, the caster, an uninvolved second character directly on the intended pull path and the Hook target beyond them, with a valid Hook landed. The target takes normal damage and is pulled, the middle body doesn't intentionally block the authoritative pull, there is no contact damage and no wall-style stop. **The endpoint isn't pinned,** because ordinary body contact is accepted. The pull must finish with no embed in a character or geometry, no Movement Validation correction, no snap-back, no prolonged stall, no oscillation and no unintended damage. **If displacement occurs, `HookPull` is still emitted,** and its `studs=` reflects the authoritative decision and isn't corrupted by client body contact. No character-sweep blocking or collision redesign is introduced.
   - **Mechanical defects found are fixed in Ticket 20** and the affected check rerun. Balance, polish and preference findings are recorded for the run-test review without expanding the ticket.
8. **Specs.** Fakes and log formats, in the existing spec style, written with the instrumentation.
   - **Pacing:**
     - the first Equip at 38.2 gives `firstAbility=38.2` and `filled=1/2`
     - the second Equip at 94.7 gives `fullKit=94.7` and `filled=2/2`
     - a later Upgrade or Replace changes neither
     - a player eliminated before any Ability has both `none`
     - a player with one Ability has the first set and the second `none`
     - a leaver keeps the values already captured
     - Reference mode produces no new pacing fields or logs
     - the `Pickup` seconds, the `MatchResult` milestones, the `Fight at`, `HookPull at` and `CacheBroken` all use the same Running-start clock
   - **Central fights:**
     - r 49.9 is `central=yes`, and `<= 50` is the pinned boundary
     - r above 50 is `no`
     - Y differences don't affect it
     - the fight keeps its tag after later hits elsewhere
     - a second fight after a timeout is classified afresh
     - the Match counters increment once per fight
     - Reference fight logging is unchanged
   - **Guard:** a successful Guard increments `guards`, a rejected one doesn't, absorbed Shield accumulates Match-locally, `MatchResult` captures both, and the next Match starts from zero.
   - **Hook:**
     - a successful nonzero player pull writes the recent-pull state
     - zero displacement and Sentry or Cache hits don't
     - a second Hook replaces the first
     - an elimination under 5 s logs the elapsed time and the caster, and over 5 s logs `none`
     - a valid player killer stays the official killer despite a recent Hook
     - the state clears across the Match and Character lifecycle
     - one `HookPull` line per successful player displacement
   - **`CacheBroken`:** `kind=Outer|Central` is reliable.
   - **Test aids:** named so they can be searched for, deleted afterwards and never committed. The word `TEMPORARY` isn't searched repo-wide, because these docs contain it.
   - **Roles:** the main session builds the `Session`, `Match` and `Combat` instrumentation and reviews everything. `studio-implementer` takes the specs from a written list, and `studio-tester` runs the Play checks, one agent in Studio at a time.
9. **Done when:**
   - the instrumentation and its specs are built, and `RunAll` passes with the new count
   - the sound pass is done, and Run R and Run M have passed on the same code, including the security and robustness matrix and its record
   - the scripted pacing floor and the central pass are recorded in a table
   - the Hook visibility and body-on-the-line leftovers are closed
   - the **Run test switch** section and both pre-publish checklists are in the ticket, and `code-structure.md` and `studio-mcp-quirks.md` carry anything new
   - the candidate meets the definition above, with `startingKit = "Reference"` and `maxSeconds = 480`
   - nothing was published or uploaded, nothing from "Later, after evidence from the run test" was started, and the work stops for the developer's go-ahead

**Progress, 2026-11-09.**

- **Instrumentation built** (items 2 and 4), by the main session, in `SessionRules`, `Session`, `Match`, `Pickups`, `Combat`, `Casts`, `Shields`, `KitRules` (`filledCount`) and `Tuning.session` (`centralRadius = 50`, `hookPullWindowSeconds = 5`). How it fits together is in [`code-structure.md`](code-structure.md#run-test-telemetry).
  - **The map's centre** is the flat average of the `Map` spawn pads, computed at Match start, rather than a new world attribute or a hard-coded (0, 0, −450). The pads sit on a ring around the centre, so it carries over to the Rift Crown.
  - **`fights` and `centralFights` count each player's `Fight` line,** because the existing fight definition is per player and no parallel encounter state was wanted. A duel between two players counts 2. Read the ratio, not the absolute number.
  - **The Hook pull isn't cleared at the death itself,** because the elimination that reads it follows the death. It clears at the elimination, a fresh character, Return to Hub, and the start and end of a Match. The 5 s window and the Match id stop a stale pull from naming anything.
  - **The session `Summary` line is unchanged:** `guards` appears only on `MatchResult`, so Reference mode logs nothing new.
- **Specs** (item 8), by `studio-implementer` from a written list and reviewed by the main session: a new `SessionSpec` (17 tests on a fresh `Session` in manual mode) and additions to `SessionRulesSpec` (7), `CombatSpec` (3), `CastsSpec` (3), `KitRulesSpec`, `CachesSpec` (`kind=Central`) and `MatchSpec` (1 each). `RunAll` on the Server in Play: **427 passed, 0 failed**, up from 394.
- **Sound pass, 2026-11-10** (item 7). The developer listened in solo Play with the Reference Kit temporarily set to Rift Hook and Stone Guard, which was restored afterwards. All four cues loaded (`IsLoaded`) and all four are **kept**: Hook cast (`swordlunge.wav`), Guard activation (`metal.ogg`), absorbed hit (`hit.wav`) and break (`glassbreak.wav`). Nothing was replaced or uploaded. For the run-test notes, not fixed:
  - **The hit sound is a bit weak.** The developer noted this as acceptable.
  - **Dash's sound (`action_get_up.mp3`) is quiet.** At first the developer heard nothing on a Dash, then heard a quiet sound. It loads and the code plays it on every local Dash. It's an older cue, outside Ticket 19.
- **Run R solo, 2026-11-10,** by `studio-tester` and reviewed by the main session: **10 of 10 pass**.

  | Check | Result |
  |---|---|
  | `RunAll` | 427 passed, 0 failed |
  | Progression dormant | No Match attributes, `MatchState`, Caches, Sentries, Pickups, or Match or loot lines. Arena pad, and the Kit is Tidal Bolt and Arc Burst at Rank 1. |
  | Basic Attack | Damage 6 |
  | Tidal Bolt | A tap did 12.0 and a 1.3 s Charge did 15, and Wet was applied |
  | Wet → Arc Burst | `WetReaction`, damage 30 (14 + 16) with `reactionAppliedBy` set, and Wet consumed |
  | Dash | A real Dash moved 21 studs and was accepted. `NoCharge` came back with charges 0 and the client reconciled. A Dash after the recharge worked. |
  | Evade | A server-timed Basic Attack into a forged Dash was `Evaded` with no damage. The control did 6. |
  | Charge interruption | Ended by a Knockback hit and by a Hook pull. The Charge attributes cleared, there was no Cast, and the later release was `StaleAttempt`. |
  | `CastRejected` | `OnCooldown` with `cooldownRemaining` 0.4, and no effect |
  | Telemetry silent | No `HookPull`, `MatchEnd`, `MatchResult` or `Eliminated`. `Fight` had no Match fields, and `Summary` had no `guards`. |

  - **Forging:** the combat checks used forged remotes from the client, and only the Dashes were real input. Feel wasn't judged.
  - **A Hook pull ends a Charge with `cause=Knockback`.** That's by design: forced movement travels as a Knockback with its own duration.
  - **Cooldown rejection was exercised only with Basic Attack.**
  - **Clean afterwards:** the helper was deleted, and the Config is still Reference and 480.
- **Two sound changes the developer asked for, 2026-11-10,** after Run R solo, both in `Tuning.feedback.sounds` and both cosmetic and client-only, so Run R isn't rerun for them:
  - **`ReactionConfirm`** (lightning on a Wet target) drops from volume 1.0 to **0.6** and speeds up from 1.1 to **1.5**.
  - **`Kill`** changes from `impact_explosion_03.mp3` at speed 1.7, which sounded too much like the Reaction, to **`victory.wav` at speed 1.25**, volume 0.8. The developer chose it by ear in Play, from 11 built-in candidates heard through a temporary audition in `Feedback` that has since been removed.
  - Nothing was uploaded.
  - **Quirk:** sounds played by `execute_luau` on the client reported `IsPlaying` but the developer heard nothing. The same sounds were audible through the game's own `Feedback`. Audition sounds in game.
- **Run R two-client, 2026-11-10,** by `studio-tester` on a developer-started Server & Clients session, and reviewed by the main session: **9 of 9 pass**, with player against player. **Run R has passed.**

  | Check | Result |
  |---|---|
  | Dormant | No Match attributes. Both Kits are Reference at Rank 1, on Arena pads. |
  | Basic Attack | 6, and the `CombatResult` reached both clients |
  | Tidal Bolt tap | 12, with Wet and a push of about 5.4 studs. The other client got `CastStarted` and drew the Cosmetic Projectile. |
  | Wet → Arc Burst | `WetReaction`, 30, with Wet consumed and `reactionAppliedBy` set to the caster |
  | Charge Telegraph | The other client saw `ChargingAbility`, `ChargingRank` and `ChargingSince`, and drew the `ChargeOrb`. WalkSpeed went 16 → 12 (×0.75) and back. |
  | Evade | `Evaded` against a real player's hit, with no damage |
  | Kill and respawn | The `Kill` line named the killer, and `killRecovery` was 0 because the killer was at full Health. The victim respawned in the Arena with the same Kit, with no Hub and no `Eliminated` or `MatchEnd`. |
  | Charge interrupted | A player's Tidal Bolt ended a Charge with `cause=Knockback`, and there was no Cast |
  | Telemetry silent | No Match lines. `Fight` lines had no Match fields, and their `opponents` named the other player. |

- **Run M part 1, 2026-11-10,** with three clients and `startingKit` temporarily `"Empty"`, by `studio-tester` and reviewed by the main session. It took three Matches, each starting with the full roster of three. **The Winner, Timeout and leave paths all pass.**

  | Check | Result |
  |---|---|
  | Start | The countdown started from the Hub, with distinct `MapSpawn` pads, 13 Caches, 3 Sentries and no loose Pickups. `MatchStart` and `LootDeal … variety=ok` were logged. |
  | PvE and loot | Caches were broken with real Basic Attacks. There was an Upgrade (Tidal Bolt and Rift Hook to Rank 2), a `ReplaceRequest` replacement, and Sentry shots taken. `Pickup` lines carry `seconds` and `filled`, and `CacheBroken` carries `kind=Outer`. `kind=Central` is covered by `CachesSpec`, and the pacing central pass will break a central Cache. |
  | PvP and elimination (Match 1) | Hook pulled 16.5 studs, then Basic Attacks finished the target. The `Eliminated` line has `by=player`, the whole Kit dropped at the body, the body was removed, the banner and Spectate showed, and `ReturnToHub` landed in the Hub. |
  | No-killer Hook elimination | A void fall 3.8 s after a pull logged `by=none hookPulledAgo=3.8 hookBy=player:-2`. One 28 s after a pull logged `hookPulledAgo=none`. |
  | Timeout (Match 2) | It timed out by itself at 480 s with all three Alive: `cause=Timeout winner=none alive=3`. All three were `Survived place=none`, and the next Match started from a clean Hub. **No shortened `maxSeconds` was needed.** |
  | Leaving (Match 3) | A Kick with two Alive gave `cause=Left`, and the Kit dropped before cleanup (`fromElimination`, present during Results). There was no double elimination, the remaining player won, and the leaver's `MatchResult` row survived. |
  | Results and cleanup | Every `MatchResult` carries `firstAbility`, `fullKit`, `guards` and `shieldAbsorbed`, consistent with the `Pickup` lines and the Guard Cast. After Results there were 0 Caches, Sentries and Pickups, and players were back in the Hub. `Fight` lines carry `match`, `at` and `central`. |

  - **Natural pacing, not a measurement:** first Ability 130–160 s and full Kit 183–202 s in Match 1, and first Ability 28.9 s in Match 3. These include tool latency and test setup.
  - **`fights=0` in Match 3 is right.** Its five Hooks were 20–60 s apart, each a lone hit that went quiet, which isn't a fight under the existing rule.
  - **Body on the pull line.** A bystander directly on the line is hit by the Hook projectile instead of the target, because the projectile hits the first character on its path. That's correct and not a pull defect. With the bystander 2.6 studs off the line, the target was pulled past them: 18.6 studs on the client against `HookPull studs=18.4`, over 0.25 s, with a maximum step of 1.23 studs per frame. There was no correction, snap-back or oscillation, and the bystander took no damage. Every sampled pull matched its `HookPull` to within 0.3 studs, and none embedded. **The leftover is closed.**
  - **Hook visibility isn't settled by the MCP.** The projectile lives about 0.14 s, about 2 frames at 15 fps, and the pull beam about 0.26 s. `screen_capture` snapshots at the start of its call, so it never caught either. The objects exist and are separate from Tidal Bolt's and Arc Burst's. **The developer judges visibility by eye** in a later session.
  - **Not exercised in Play:** `shieldAbsorbed > 0`, which is covered by the specs. The Guard in Match 1 had expired before any hit.
- **Hook visibility, 2026-11-10:** the developer looked at Rift Hook on `MatchMap` in a three-client session and judged it good. No tweak was made. **The leftover is closed, and so are all of Ticket 19's leftovers.**
- **Flood guards, checked in the code by the main session:** only `ReplaceRequest` is guarded, with a token bucket of burst 5 and 5 per second, which drops excess before validation. `CastRequest`, `ChargeStart`, `ChargeRelease` and `DashRequest` have no guard, and each malformed request prints one unthrottled warning. `ReturnToHub` has no guard, but its warnings are throttled to one per second per player.
- **A test aid for the security matrix and the pacing runs:** `ServerScriptService.Server.TEMPORARYRunMHelper`, with `TEMPORARYRunMHelperFn` in ServerStorage. It grants an Ability with `Kits.grant`, which logs no `Pickup` line, drops a Pickup and reads a Kit. It isn't committed anywhere, and it's deleted before the candidate.
- **Run M part 2, the security matrix, 2026-11-10,** with three clients, by `studio-tester` and reviewed by the main session. **All five pass criteria are met, and there are no findings.**

  | Remote | Guarded | Malformed | Flood | Integrity and recovery |
  |---|---|---|---|---|
  | `ReplaceRequest` | Yes, a bucket of burst 5 and 5 per second | String, non-integer, NaN, infinity, extra `abilityId` or `rank`, a missing field, or a non-table payload: dropped `malformed`. Negative or unknown id: `Gone`. Slot 0 or 3: `NoSuchSlot`. | 50 in one tick on distinct Pickups: exactly 5 reached validation. 50 identical: 1 success. 45 over 4.97 s: 29 successes, where the bucket allows about 29.8. | No duplicate mutation and one outgoing drop. A legitimate request worked 0.1–0.5 s after a flood. |
  | `ReturnToHub` | No. Warnings are throttled to 1 per second. | Extra field or non-table: `malformed` | 50 repeats after a success: harmless, with no duplicate spawn or log line. Heartbeat max 23.9 ms. | Alive: `Alive`, stays in. Hub: `InHub`. |
  | `CastRequest` | No | (as Ticket 18) | 50 in one tick: 1 accepted, the rest refused. dt mean 16.7 ms, max 18.4. 0 warnings. | Works straight after |
  | `DashRequest` | No | (as Ticket 13) | 50 in one tick: 1 accepted, 49 `AlreadyDashing`. Max 17.7 ms. | Works straight after |
  | `ChargeStart` | No | (as Ticket 13) | 50 in one tick: 1 Charge, 49 `Charging`. Max 17.6 ms, and 24.6 ms once afterwards, which recovered at once. | A tap Charge works straight after |

  - **`ReplaceRequest` state gates,** each refused with no outgoing drop, the Pickup kept, and the Kit and Equip Lock unchanged:
    - `TooFar`
    - `Charging`
    - a Kit that isn't full: `SlotEmpty` or `Fits`
    - Eliminated, Hub, Results and Countdown: all `NotInMatch`
  - **Race,** twice, with the two requests landing within about 2 ms of each other: exactly one success, the Pickup removed once, one outgoing drop, and the loser got `Gone` with its Kit unchanged. The same player won both times. That isn't pinned, and nothing depends on it.
  - **Gates outside active play,** from Eliminated, Hub, Results and Countdown: Basic Attack, all four Abilities, Charges and an unknown Ability were all refused `NotInMatch`. Health, Shield, Charging and the Kit were unchanged, and a target 8 studs away kept 100 Health. `DashRequest` was accepted in the Hub, which is by design. From an Eliminated player it was `NoCharacter`, with no mutation.
  - **Cache damage:** the control did 75 → 69. From an Eliminated player beside the Cache and from the Results winner, 4 and 9 Basic Attacks were all `NotInMatch`, with Health and `CacheStage` unchanged, no Pickup and no `CacheBroken`.
  - **Covered by the code rather than Play,** as reviewed by the main session:
    - The Waiting phase takes the same `phase == "Running"` gate as Countdown and Results.
    - A voluntary Hub spectator is `Hub` on the server, because spectating is client-only.
    - An Eliminated player's in-range `ReplaceRequest` is refused by `canCollect` before the range check.
    - Hub-player Cache damage in range can't happen, because the Hub is far from every Cache.
  - **Observations, not findings:**
    - An outgoing drop lands within 4 studs, so it was collected back as an Upgrade during the floods. That's the known Ticket 19 quirk.
    - Throttled warning counters carry over between Matches.
    - Server-side errors came only from the tester's own snippets.
- **Run M part 3, pacing, 2026-11-10,** with three clients walking at once, by `studio-tester` and reviewed by the main session. Every value is **floor pacing under perfect route knowledge**: it excludes search, hesitation, reading Pickups, loot decisions, avoiding players, aim and detours. The times come from the server's `Pickup` and `MatchResult` lines.

  | Match | Player (pad) | First Ability | Full Kit | Caches |
  |---|---|---|---|---|
  | 2 | P1 (1) | 7.8 | 17.0 | Outer01, Outer10 |
  | 2 | P2 (6) | 7.6 | 16.7 | Outer09, Outer08 |
  | 2 | P3 (3) | 7.5 | 16.7 | Outer04, Outer03 |
  | 3 | P1 (4) | 7.9 | 17.1 | Outer05, Outer06 |
  | 3 | P2 (1) | 7.9 | 17.1 | Outer01, Outer10 |
  | 3 | P3 (5) | 7.7 | 16.9 | Outer07, Outer08 |
  | 4 | P1 (4) | 7.8 | 17.1 | Outer05, Outer06 |
  | 4 | P2 (1) | 7.8 | 17.1 | Outer01, Outer10 |
  | 4 | P3 (6) | 7.6 | 16.8 | Outer09, Outer08 |

  - **Outer, over the 9 required runs:** first Ability min 7.5, median 7.8, max 7.9. Full Kit min 16.7, median 17.0, max 17.1. An extra Match 5 gave 7.0–8.5 and 16.4–17.8. There was no disruption, no duplicate, so a third Cache was never needed, and no shared Cache.
  - **Where the time goes in one Cache cycle of about 8.5–9 s:**
    - walking 2.0–2.8 s
    - **breaking 5.2 s** (13 Basic Attacks at 0.43 s)
    - collecting 0.3–0.6 s

    The walker's own 0.7 s pause after collecting inflates the full Kit slightly, so the true floor is about 16 s.
  - **Central pass** (Match 8, pad 1 to `CacheCentral1`, Sentries active): a waypoint is needed around the core, and the walk took about 7 s. The break took about 8.3 s (20 Basic Attacks) and `CacheBroken` came at 15.5. First Ability 15.9 and full Kit 16.7, both from that one Cache. Three Sentry shots of 12 each, about 3 s apart, pushed the walker about 5 studs each. Health never fell below 76, and the pass was neither interrupted nor slowed. Two earlier attempts were disrupted and not counted.
  - **Movement Validation:** real-speed `MoveTo` caused no violation, snap-back or speed correction, and the server logged no warning.
  - **A harness observation, not a defect:** after a Sentry hit, one `MoveTo` walker crawled at 0.5–2 studs/s for minutes, with `MoveDirection` 0, `WalkSpeed` 16 and nothing attached to it. `Humanoid:Move` on the same character gave 13–16 studs/s. Real player input drives `Move`, so players aren't affected. The central pass used `MoveTo` plus a Heartbeat `Humanoid:Move` driver, which is the ticket's second fallback: continuous movement at real WalkSpeed, with no hops.
  - **Against the references.** The floor matches Ticket 18's hypothesis for the first Ability (8–10 s), and is faster than its full-Kit estimate (about 25 s), at about 17 s. Milestone 2's design targets are a first Ability at about 30–45 s and a full Kit at about 1.5–2 minutes. The floor is about 4–6× faster for the first Ability and 5–7× faster for the full Kit. The central route costs about 2× an outer Cache to a first Ability, but yields both Abilities at once, so a full Kit from the centre arrives at the same time as from the outer route. In the run tests that ran naturally (Run M part 1), with tool latency and setup included, a first Ability took 29–160 s.
  - **Recommendation: no config change.**
    - The human run test is decisive, and a floor that beats the targets isn't a reason to tune by itself.
    - The time is mostly in breaking, not walking. Cache Health is the obvious lever, but it would also raise the central cost, which is currently the reason to go central at all.
    - Ticket 21 replaces this map with the Rift Crown and sets a new baseline, so tuning `MatchMap` now would be wasted.
    - Carry these numbers into the run-test review. If testers also reach a full Kit well under a minute, revisit loot density on the Rift Crown, not Cache Health on this map.
- **Reference-safe publish candidate, 2026-11-10.** It's the Studio state at the commit that records this entry. It isn't a Roblox version, and nothing was published or uploaded.
  - **`RunAll`** on the Server in a fresh solo Play: **427 passed, 0 failed**, with the telemetry specs included. There were no errors and no `MatchPhase`.
  - **Run R and Run M passed on the same code.** The only changes after Run R were the developer's cosmetic, client-only sound tweaks, as version discipline allows.
  - **No temporary aid remains.** `TEMPORARYRunMHelper` was deleted, and its BindableFunction only ever existed at runtime. A case-sensitive search of every instance name and every script's source for `TEMPORARY` finds nothing.
  - **Tuning is restored:** `Tuning.kit.startingKit = "Reference"`, `reference = { "TidalBolt", "ArcBurst" }`, and `Tuning.match.maxSeconds = 480`.
  - **The git tree is clean** at that commit.
  - **Changed since version 86:**
    - the run-test telemetry: `Session`, `SessionRules`, `Match`, `Pickups`, `Combat`, `Casts`, `Shields`, `KitRules.filledCount`, and `Tuning.session.centralRadius` and `hookPullWindowSeconds`
    - its specs: a new `SessionSpec` and additions to six others
    - the developer's sound changes: `Kill` is now `victory.wav` at 1.25, and `ReactionConfirm` is volume 0.6 at speed 1.5
  - **Findings:** none unresolved. The observations for the run-test review are:
    - the hit sound is a bit weak and Dash is quiet
    - floor pacing is far ahead of the design targets, with no change recommended
    - the `MoveTo` crawl, which affects only the test harness

  **Handoff:** the candidate is a clean commit with 427 specs passing and `startingKit = "Reference"`. Run R and Run M have passed and nothing is unresolved. **Publishing is manual**, using the Reference / Stage 3 checklist in item 6. To prepare the progression run test afterwards, follow the **Run test switch** in item 6. Record each resulting version in Checkpoints with its commit.

**Ticket 20 is done,** waiting on the developer's publish and go-ahead.

### 21. Rift Crown Arena layout

**Status: built and measured on 2026-11-11, with 498 specs passing. Human play checks postponed by the developer (2026-11-11).** Measurement rows 1 to 7, the two-client Match, the Void kill with credit (Tidal and Hook) and the Kit drop are done and recorded below, and the developer's decisions on the findings are settled. The play-check table (2, 4 and 6 players, seven in the Hub) is **postponed**, to be run later or by a smaller test audience. The Match 1 `Died by=none` fall is open. See the [build record](#build-record-2026-11-11-steps-1-to-4-not-complete). Specified 2026-11-09 and reviewed adversarially the same day. Every design point was grilled with the developer and settled. The reasoning, rejected options and edge cases are in [`rift-crown-design.md`](rift-crown-design.md); this entry is the build contract. Numbers are labelled **fixed** (a locked decision), **tunable** (a starting value that lives in `Config` and can change without a redesign) or **greybox-validated** (it depends on a measurement or a geometry check, so it is confirmed or changed in Studio before the ticket is done).

**Expected values are not pass criteria.** The measurement table separates what we *expect* from what *passes*. A measurement that lands outside its expected value triggers the adjustment named beside it. It does not fail the ticket unless the row has an explicit threshold.

#### Summary

Replace the greybox `MatchMap` with the first real Arena, the **Rift Crown**: a ruined citadel shaped like a shattered crown, suspended over the Rift. It is one central **Crown** and six outer **Shards**, joined by typed bridges, with an open perimeter and a physical void below. The ticket builds the map, its markers and metadata, the spawn and arrival rules, the void elimination with kill attribution, and the structural validator. It does **not** build the Collapse, Wardens, XP or any new hazard, but the map carries the data they need.

#### Scope

- The Rift Crown geometry as authored Parts and Models, built as `Workspace.RiftCrownBuild` and swapped in as `Workspace.MatchMap` (see Build and swap strategy).
- **Cache kinds `Home`, `Forward` and `Crown` replace `Outer` and `Central`** in `Tuning.caches`, `Caches`, `LootRules` and the specs. Values are carried over, not retuned: Home and Forward use the Outer health (75) and 1 drop, Crown uses the Central health (120) and 2 drops.
- The 24-drop loot layout and its **Ability deal**, and 3 Crown Sentries on the existing Sentry system (its values are unchanged).
- Six spawn pads, the **spread-table assignment**, the **roster cap** and the assignment backstop.
- The **arrival hold**, `goAt`, and the 480 s Match clock starting at GO (Ticket 22 raises `maxSeconds` to 600; the clock origin stays GO).
- The **Crown seal** (`crownOpenSeconds`), implemented and tested here, not only prepared for.
- **Void elimination** at `voidKillY`, cause `Void`, displacement credit and the inward Kit-drop offset.
- Collapse-ready metadata and the **structural validator**, which only reads data. No Collapse behaviour.
- Telemetry for the opening, the Crown and pacing, and a **part-count baseline**.
- Updating the affected specs: `MatchMapSpec`, `CachesSpec`, `LootRulesSpec`, `MatchRulesSpec` (its wrap-around test changes) and `MatchSpec` (the fall test at `Removed`).

#### Non-goals

- **Rift Collapse.** Ticket 21 builds and validates `SectionId`, `CollapseLayer`, `CollapseParent`, `ShardId`, connector metadata, slot `SectionId` and `CollapseCenter`, and nothing more (amended 2026-11-10: plus the presentation metadata set out under [Presentation metadata for Ticket 26](#presentation-metadata-for-ticket-26-added-2026-11-10), which adds no gameplay scope). It does **not** remove sections during play, damage players with the Rift, time Collapse phases, add Collapse UI, resolve loose Pickups on collapsing floor, decide what Crown Sentries do late, or decide Dais overtime.
- **Rift Wardens.** Ticket 21 provides only reserved slots, spawn-distance compatibility and section metadata. It does **not** build Warden AI, aggro, attacks, XP, loot or an elite. The roughly 25-stud aggro range from the design is an example, not a requirement. [Ticket 23](#23-rift-warden) settles the Warden and refines these slots: a `GateId` and `TerritoryRadius` on each slot, a `GateId` on each pad, and a preferred +15 to +20 degree offset toward the Gate seam (the +10 degree position is the fallback).
- XP, levelling, equipment or Relic slots, new Abilities, loot tables beyond the 24-drop layout.
- Any active environmental hazard, hazard anchors or scripts.
- Final colours, textures, meshes, particles, sounds, uploaded assets, HUD, Match Intro or polished insertion/camera/VFX. **No upload without explicit approval.**
- Free-drop or glide spawning, a Dash ledge guard, fall recovery, a Fall state, hidden edge-kill volumes.
- Player-count map variants, a multi-map framework or a procedural map framework.
- Combat redesign. Cache health, Sentry fire rate and Ability values are not changed.

#### Settled topology

- **1 Crown + 6 Shards**, 7 fragments, 1 spawn per Shard (**fixed**). The map is the same for 2–6 players. Unused Shards stay fully playable and lootable.
- No Shard is a safe island. Every Shard has a Crown spoke and at least one non-Dash rim route.
- Radial: the future Collapse consumes Shards → Approach → Crown.

#### Fragment structure and dimensions (**tunable** unless noted)

| Item | Value |
|---|---|
| Arena diameter | about 260 (r ≈ 130 outer lip), centre at the origin of the map, **fixed size** |
| Crown | about Ø64 (r 32) |
| Shard | about 80 wide × 82 deep, irregular, usable area 4,000–6,000, individual Shards ±20% |
| Crown to Shard gap | about 16 |
| Shard to Shard rim gap | 10–24 |
| Shard centres (azimuth) | 30°, 90°, 150°, 210°, 270°, 330° (centre at r ≈ 89) |
| Shards differ | in silhouette, edge profile, local ruin layout, **not** in role |

**Target walk times at 16 studs/s (greybox-validated):** spawn → first connector 2–3.5 s; spawn → Crown about 6 s; neighbouring spawns 8–9 s (about 12–13 s where the route is a Dash pair and Dash is saved, deliberately); opposite spawns about 17 s; full traversal about 16 s.

#### Crown specification (topology and azimuths **fixed**, dimensions **tunable** and greybox-validated)

- Terrace r 14–32 at y = 0. **Dais** r ≈ 14 raised about +5, **empty in v1**, reached by three ramps at **60°, 180°, 300°**. With the Rib at r ≈ 24 each ramp's run is only about 10 studs, so a ramp is about 27° steep. That is a deliberate exception to the 15° terrain guideline (the 5-stud face may also be jumpable, to be measured). Walk and fight on a ramp in the greybox.
- **Three Ribs** (curved walls, about 14 long, about 8 high, line-of-sight blockers, not platforms) at 60°, 180°, 300°, r ≈ 24, just outside each ramp foot.
- **Three Cache/Sentry bays** at **0°, 120°, 240°**: Crown Cache at r ≈ 19, Sentry at r ≈ 27 on a 2–3 high plinth behind it. Drops land about ±5 studs across the bay.
- **Sentry buttress (settled in review, confirmed by the developer).** A Sentry at r 27 sits 5 studs from the lip and its push goes away from the Sentry, so a player standing behind it would be pushed over the rim, which the Point 3 rule forbids. *The Sentry bay geometry must convert the Sentry's outward displacement into safe collision or containment rather than a cheap void kill, while leaving the bay readable and combat-relevant.* Each Sentry plinth is part of a masonry **buttress** behind and beside it, in the Shattered Crown kit. It must:
  - be enough geometry to catch the expected push trajectories;
  - **not** create a fully enclosed dead-end pocket, block the normal bay approach, or give the whole bay unexpected Hook collision;
  - **not** block the Sentry's intended line of sight toward the playable bay;
  - **not** create a hiding pocket where a player can permanently escape Sentry or PvP interaction.

  Exact dimensions are **greybox-validated**, because they depend on the measured Tidal tap and Sentry push distance.
- **Sentry sweep check (acceptance, Edit-mode raycast script).** At minimum it proves: (1) from every intended player-standing position in the bay, the expected Sentry knockback trajectory meets safe Crown geometry before the lethal rim; (2) no intended standing position is a one-shot Sentry-to-void setup; (3) the buttress does not block the Sentry's target lane to those positions. If a measured push is larger than the margin, change the buttress or the Sentry position, not the rule.
- **Lip rule:** no Cache, drop point or Sentry-covered standing spot within about 10 of a lethal lip (**tunable**, subject to the Tidal measurement below).
- **Combat footprint:** the Inner Crown's about 1,800 studs² is the disc r ≤ 24 including the Dais top (about 1,190 terrace ring + 620 Dais). The terrace ring alone is about 1,190. Plan final fights with that in mind.
- The Crown is not a required source of a first functional Kit.

#### Connector graph and route taxonomy

All six Shards connect **directly to the Crown**, one landing each, at **40°, 80°, 160°, 200°, 280°, 320°** (**fixed**). Three **Gates**, each around one Rib, pair two Shards.

| Gate (rib) | Safe spoke | Risky spoke | Landings |
|---|---|---|---|
| 60° | Shard 30° → 40° | Shard 90° → 80° | 40°, 80° |
| 180° | Shard 150° → 160° | Shard 210° → 200° | 160°, 200° |
| 300° | Shard 270° → 280° | Shard 330° → 320° | 280°, 320° |

**Rim links (between Shards):** Safe pair links 30↔90, 150↔210, 270↔330; one Risky link **210↔270**; **Dash shortcuts** 90|150 and 330|30 (marker/data objects, not geometry).

**Radial placement (needed for the layer partition, greybox-validated):** the three Safe pair links sit in the inner half of the rim gap (about r 55–80, Layer 2). The Risky link and the Dash shortcuts sit toward the outside (about r 95–120, Layer 1). The rim gap is a wedge that widens outward, so the gap bands below apply at the chosen crossing point, not everywhere along it.

| Type | Width / gap | Rules |
|---|---|---|
| **Safe** | 14–18 wide (**tunable**) | parapet about 4 high, 1.5–2 thick (**greybox-validated**), continuous, open at aprons, doubles as low cover |
| **Risky** | 8–10 wide (**tunable**) | no parapet, no side geometry, readable as dangerous before committing; one push is never an automatic kill |
| **Dash shortcut** | gap 10–14 (**greybox-validated**), takeoff and landing aprons | always a non-Dash alternative; failing may mean a fall |
| Bridge-required gap | 18–24 (**greybox-validated**) | **if Dash + jump crosses it, widen toward 24** |
| Jumpable gap | up to about 6 (**greybox-validated**) | not in the macro graph; only local Shard geometry after measurement |
| Never | over about 30 | |

Principles (**fixed**): a Shard always has a non-Dash route; no one is stranded for spending or saving Dash; the Crown is never the only link between large parts of the Arena; most fighting happens on broad terrain, not bridges; routes read as Safe or Risky before commitment. **No Cache, Sentry, Warden or required objective sits on a connector or its apron.** Landings approach from the Rib sides and are at least 15° from any Sentry lane.

#### Verticality (the datum and the exclusions are **fixed**, heights are **tunable**)

- One datum, **y = 0** (**fixed**): Shards, decks, aprons, Crown terrace. Opposing connector edges are on the same datum. No whole-Shard elevation differences.
- Terrain relief about ±2, slopes under about 15°, no internal step over about 2 without a ramp.
- Ruin platforms **+3 to +4**, 1–2 per Shard, 14–20 across, at least two accessible sides, at least 10 from a lethal lip. Low cover 3–4, plinths 2–3. The reach checks below may change these heights (adjust local geometry first).
- **Not in v1** (**fixed**): multi-level Shards, caves, ladders, undercrofts, lower routes, overpasses.

#### Spawn placement, spread assignment and the roster cap

- Six invisible setup pads (existing `SpawnLocation`s under the map folder, `Enabled`, transparent and non-colliding), one per Shard, at the Shard's centre azimuth, **r ≈ 100** (**tunable**), facing the Crown. Pad rules (**greybox-validated**): lip margin 25–30 or more, no direct pad-to-pad sight line, first connector in 2–3.5 s. A pad may shift if local geometry helps.
- **Spread table** with a **random rotation** per Match (none favoured): 2 → opposite Shards; 3 → alternating; 4 → spread to avoid clustering; 5 → all but one; 6 → all. No wrapping, **no Hub drafting**. This replaces today's uniform shuffle in `MatchRules.assignSpawns`.
- **Primary rule: the roster is capped at six when it is formed** (`MatchRules.buildRoster` at the Countdown's end). If more than six are eligible, six are chosen at random with the injected `Random`, and the rest **stay Hub players, still eligible for the next Match** (nothing is dropped or flagged as an error). Seven players in the Hub must still produce a six-player Match.
- **Defensive backstop: `assignSpawns` refuses and logs** when given more ids than pads, or an empty pad list. It never wraps onto an occupied pad. The check runs in the pre-start step that already deals the loot, so a refusal disables Matches with a reason and leaves the world untouched (as `disableMatches` already does for a bad loot deal), not after `startMatch` has started changing it.
- `Characters.padsFor` reads the pads once per server, so a swap takes effect in a new Play session.
- No aggro-capable PvE within `pveSlotClearance` of any pad (see PvE slots).

#### Arrival hold and GO

- The server places every roster character on its pad and holds them for **`arrivalHoldSeconds` = 3** (**tunable**, 0 disables). `goAt` is set when the last roster character has been placed. A character that cannot be placed goes through the existing loss path (`Removed`) and does not extend the hold.
- **Disabled before GO, server-enforced:** movement (frozen by the server, not just ignored by the client), Basic Attack, Dash, casts, Cache damage, Pickup collection and replacement, Sentry targeting, PvE aggro and combat. **Allowed:** camera look and the leave/Return-to-Hub path.
- **At GO:** controls, combat, Sentries and PvE become active, and the **480 s Match clock** and all pacing telemetry start. Today `startedAt` is set at `MatchStarted`, so the builder must separate "Match entered" from "GO". The seal clock is also measured from GO.
- **During the hold:** the phase is still Running, `MatchRules.canAct`, `canCollect` and Sentry targeting also check `goAt`. A player who leaves is eliminated as `Left` as usual, and if that leaves one Alive from a roster of two or more the Match ends normally. `Void` does not apply.
- Caches near a pad may be visible with contents unknown. Presentation length must not silently change gameplay timing.

#### Loot and PvE spatial slots

**24 Ability drops** (**fixed** for this ticket, the economy itself is tunable later): 12 Home + 6 Forward + 6 Crown.

| Slot | Count | Placement (**greybox-validated**) | Rules |
|---|---|---|---|
| Home Cache | 12 (2 per Shard), 1 Ability each | 35–50 from the pad, 15+ from a lethal lip | no Sentry, no mandatory PvE |
| Forward Cache | 6 (1 per Shard), 1 Ability | the Forward Cache rule below; **about 50–58 from the pad is a tunable starting target, not a pass band** | not on a bridge, apron or lip; about 50 from Crown Sentries |
| Crown Cache | 3 (1 per bay), 2 Abilities each | r ≈ 19 in the bays | sealed, then opens |
| Sentry | 3, Crown only, active from GO | r ≈ 27 on plinths | no Shard Sentries in v1 |
| Warden slot (reserved) | 3 | inner corner of the Safe-spoke Shards (30°, 150°, 270°), near the start of the spoke, not on the connector | hypothesis, relocate if Safe routes become systematically worse |
| Elite slot (reserved) | 1 | Dais | nothing spawns in v1 |

- **Forward Cache rule (settled in review, confirmed by the developer).** *A Forward Cache must be clearly farther from spawn than the Home Cache pair, positioned toward the Gate / contested side of the Shard, while respecting the lethal-edge margin and remaining fully inside valid Shard geometry.* The old 60–75 studs from the pad did not fit an about 80 × 82 Shard with a 15-stud lip margin (the farthest valid point toward the Gate is about 50–58), so **the structural rule takes priority over the number**. The Home band (35–50) is unchanged. About 50–58 from the pad is a tunable starting target. If one Shard's geometry needs a Forward Cache slightly outside it to meet the rule cleanly, keep the rule. The Forward Cache must not overlap the Home Cache band and must not sit on a connector or apron.
- **PvE spawn-safety rule (fixed form):** an aggro-capable PvE slot is at least `pveSlotClearance` from every pad, **including its own Shard's pad**. `pveSlotClearance` is a planning constant (**tunable**, 45, which is the hypothetical 25-stud aggro range plus a 20-stud buffer), used only by the validator. The real Warden aggro range is not locked here. The Warden slot at a Safe-spoke Shard's inner corner is tight against this, so if it does not fit, move the slot, never the pad rule.
- **Ability deal (settled in review).** The deal must hold these invariants for any seed:
  1. Exactly `drops / pool size` copies of each Ability (6 each for 24 drops and 4 Abilities).
  2. **Each Shard's three local drops (Home A, Home B, Forward) are three different Abilities.**
  3. The Crown Caches hold exactly the remaining 6.
  4. No Crown Cache holds the same Ability twice.
  5. Crown Ability counts differ by at most 1, so the Crown is not skewed toward one Ability.
  6. Which Shard gets which triple, and which of its three drops is Home A, Home B or Forward, is random per Match. No physical Shard is tied to a pattern.
  7. Consequence: **no Shard offers a duplicate, so Rank 2 is impossible from a Shard's own drops.**
- **Reference algorithm for the current 4-Ability pool** (verified; the spec checks the invariants, not this algorithm). Each Shard omits exactly one Ability and takes the other three. Let `o(X)` be the number of Shards that omit X. X then appears in `6 − o(X)` Shards, and the Crown bag holds `o(X)` copies, so every Ability totals 6 and the Crown holds `Σo = 6` drops. Six omissions over four Abilities cannot be equal (1.5 each), so use the balanced multiset **{2, 2, 1, 1}**: **randomise each Match**: pick at random which two Abilities are omitted twice (the other two once), then shuffle the six omissions over the six Shards. The balance {2, 2, 1, 1} is fixed, but which Abilities and which Shards get which counts is not. The Crown bag is then 2, 2, 1, 1 copies. Deal it to the three Crown Caches by listing it grouped by Ability and giving item *i* to Cache *i* mod 3, which never repeats an Ability inside a Cache (each Ability has at most 3 consecutive items), then shuffle which bay gets which. Pair partners may end up with the same triple. That is acceptable, because any two different triples overlap anyway.
- If the pool cannot meet the invariants (a pool of fewer than three Abilities, or a size that does not divide 24), the deal refuses with a reason and Matches are disabled, as `validate` already does. A larger pool needs the invariants re-verified.
- This replaces `LootRules.deal`'s Outer/Central dealing and its best-effort `nearestPairs` variety, which becomes unnecessary.
- Intended opening: spawn → first Home Cache → second → Forward Cache → functional Kit → choose rotation, Crown, PvP or Rank. Targets (**hypotheses, to measure**): full Kit about 25–35 s; Rank 2 about 60–80 s at 2 players and 70–90 s at 6.

#### Crown seal (in scope: implemented and tested here)

- `crownOpenSeconds` = **60** (**tunable**, range about 45–90), measured on the Match clock, so **from GO, not from character creation**. Before it, the three Crown Caches are **visible and their value readable, but cannot take damage, open or drop loot**. The server rejects damage at the single Cache damage entry, whatever its source.
- Each Crown Cache record owns its own `opensAt`, so staggering is possible later. **No stagger in v1.**
- At the open there is a clear world-visible and audible Rift event (a placeholder is fine, final VFX/audio is later). The server publishes it through the existing world state, in a shape that passes `Protocol.validate`, and logs `CrownOpened`.
- **Config edge cases:** `crownOpenSeconds` must be a finite number, at least 0 and less than the Match length. `0` opens at GO, so the sealed state is never seen but the event still fires. Anything else disables Matches with a reason, never a silent default. If the Match ends before the open, nothing fires.
- Sentries are active from GO, sealed or not. The seal does not depend on the Collapse.
- Structural requirement: one seal object or anchor per Crown Cache (chains + glyph) that a later presentation can fracture and release.

#### Void and elimination model

- Physical fall, and the Match loop (every `loopSeconds`) eliminates at **`voidKillY` = −40** (**tunable**, see the fall-time measurement). `FallenPartsDestroyHeight` (−500) is only a safety net. No hidden volumes, no Fall state, no recovery, no Dash ledge guard.
- **Scope:** only Alive players in a Running Match on the Rift Crown, after GO. Never in the Hub, the Reference environment, the arrival hold or Results. A fall during the end's resolution window still counts, because the phase is still Running. A player who dies by health first is a normal `Died`, and a first elimination wins if two things race (`MatchRules.eliminate` already returns false the second time).
- Elimination cause **`Void`** is added beside `Died`, `Removed` and `Left`. The consumers that list causes (the `Protocol` comment and validation, results rows, feed text, specs) must accept it.
- `voidKillY` must be below the lowest authored underside with a margin (a rule for the map, checked by the validator), so a falling player is never killed while still beside geometry.
- **Kit drop after a void death (simplified in review).** No navmesh and no edge database.
  1. Start from `lastGrounded[id]` (already a stable floor point updated every loop).
  2. Inward = the horizontal direction from the point where the player left the map (their position at the kill) back toward `lastGrounded`. If that distance is under 0.5 studs, use the direction toward `CollapseCenter`.
  3. Candidate = `lastGrounded` moved about **6 studs** inward (**tunable**).
  4. Valid if a downward ray from above the candidate hits a part of the map at about the same height (±3) as `lastGrounded`. Otherwise try half the distance, then use `lastGrounded` itself.
  5. **Each dropped item's spread point (`dropSpreadStuds`) is validated the same way**, and collapses to the origin if it is off the floor. Today the spread points are not checked, so a Kit dropped at a lip could place items over the void.
  6. Never drop into the void.

#### Environmental kill attribution

- One **displacement-credit record** per victim: the **most recent valid hostile player displacement** (Rift Hook pull or Knockback): attacker, time, source. A newer one replaces the older. There is no assist system. Starting expiry (**tunable**): cleared after about 1 s of **continuous** floor contact, hard expiry about 6 s. Any airborne frame restarts the 1 s, so a one-frame brush with a cliff, bridge side or debris does not clear it.
- A fall with a live record: **killer credit, cause `Void`**, source and the record's age kept for telemetry. No record: self-fall, cause `Void`, no killer. Records are cleared at the start and end of a Match.
- **Sentry pushes and future PvE displacement award no player kill and neither create nor clear a record.**
- Record the action at the existing hit path (`Combat`, where `applyKnockback` is called with the attacker known) and at Rift Hook's pull. Read it in the elimination path (`Session.killerOf` returns nil for falls today). Re-read both before building.

| Case | Rule |
|---|---|
| Attacker leaves or is eliminated before the victim reaches the plane | Credit stands while the attacker was on the roster. Their kill count updates even if they are Eliminated or back in the Hub. |
| Pushed by A, then by B | The most recent valid player displacement wins, B. No sharing. |
| Brief contact while falling | Does not clear the record (needs 1 s of continuous floor contact). |
| A Sentry pushes after a player did | The player's record stays and gets the kill credit. The Sentry's part is logged only. Accepted simplification. |
| Dashes off on purpose several seconds after a push | Credit stands until the 1 s grounded or 6 s expiry. Telemetry records the age so the 6 s can be tuned. |
| Character destroyed or killed before the plane | A health death is `Died` through the normal path. `Removed` and `Left` carry no credit. `Void` applies only at the plane. |
| Fall during the hold, Results or the Hub | No `Void` (out of scope). The hold cannot move anyone. |
| Fall at the same moment as the last other player's death | `MatchRules` already resolves simultaneous ends. The spec covers one case. |

#### Edge categories (the categories are **fixed**, the numbers are **tunable** and greybox-validated)

1. **Lethal edge:** open lip, sheer face of at least about 12 studs before any catching geometry, seam trim, no parapet. The danger band is **greybox-validated**: tune against the measured tap and charged Tidal pushes, and never derive it from either figure in the existing docs (see the Tidal measurement). If charged Tidal turns out to be much larger than assumed, **re-grill whether it is a map or combat issue** instead of widening every edge.
2. **Safe edge:** a drop that is not void: Dais face, ruin platforms, ramps, steps. Solid floor within about 4 studs. Plain chamfer or step, no seam.
3. **Protected connector edge:** Safe connectors only, continuous parapet.
4. **Risky connector:** exposed deck, nothing at the sides, nothing hanging beneath with collision.
- **No railings** on Shards, the Crown, Risky bridges or Dash gaps.

#### Collapse-compatible section partition

| Layer | Radius (**tunable**) | Contains |
|---|---|---|
| 1 Outer | r ≈ 85–130 | outer Shard portions, pads, most Home Caches, outer perimeter, Risky rim bridge, Dash pinch regions |
| 2 Middle | r ≈ 48–85 | inner Shard portions, Forward Caches, Warden platforms, **the three Safe pair links** |
| 3 Approach | r ≈ 24–48 | six spokes, Crown landings, outer Crown band, Sentry plinths |
| 4 Inner Crown | r ≈ 14–24 | inner terrace, Ribs, Crown Caches |
| 5 Dais | r < 14 | the Dais, **structurally supported, use by the Collapse not decided** |

- Each physical section becomes unsafe **as a unit**. The map is not a shrinking circle cutting through geometry.
- **All Shards share the layer order** (no Shard stays an outer island). Jitter is allowed within a layer, **never reordering the inward chain**.
- **Parent rule:** `CollapseParent` exists and has a strictly later layer. The route to the parent stays walkable while the section is active, needs no Dash, no lethal-gap jump, and no earlier-collapsed route. Section splits avoid cutting through ramps, platforms, Caches, Warden slots, pads and aprons.
- **Narrow-route rule:** the Collapse never forces an on-time player across a Risky connector.
- **Inward escape:** a player's inward route outlives the section they leave.
- **Geometry:** sections are real slabs about **12+ thick** with a rock/structure underside, so a removed neighbour reveals a lethal face, never a thin plate or a hole that disagrees with collision. The Dais and Ribs may extend down as pillars.

#### Section, connector and slot metadata

| Object | Attributes |
|---|---|
| Section | `SectionId` (unique), `CollapseLayer`, `CollapseParent`, `ShardId` (Shard sections) |
| Connector | `ConnectorType` (Safe, Risky, Dash), `CollapseLayer`, section/Shard association |
| Dash gap | marker/data object, not geometry |
| Cache, Sentry, Warden, spawn markers | reference a `SectionId` |
| Map | one authoritative `CollapseCenter` marker |

Exact folder layout, attribute names for Cache kinds and the marker model are an implementation decision recorded when built. **No generic hazard-marker framework.**

#### Shattered Crown identity and construction kit

*A ruined citadel shaped like a shattered crown. The six Shards are the broken points, the Crown is the circlet, the Dais the empty jewel setting.* Generic gladiator arena and natural floating landscape are rejected.

**Ten-piece kit (fixed):** 1 slab (12+ thick, chamfered playable lip, faceted underside); 2 Rift seam (lethal edges and Collapse-readable fractures only); 3 masonry; 4 column stumps; 5 arch/rib segment (signature silhouette); 6 iron bands/chains (visually distinct from Hook amber-orange); 7 parapet (Safe connectors only); 8 plinth (Sentries, Dais, future bases); 9 paving/apron (arrivals, Gate transitions); 10 crystal clusters, sparingly.

- **Shard distinction:** one landmark per Shard from the same kit, no separate biomes.
- **Sentries** read as carved guardians of the citadel, future **Wardens** belong to the same Stone/Rift family without being confused with Sentries. No monster appearance is finalised.
- **Colour:** avoid leaning on Tidal blue, Arc lightning, Hook amber-orange or Stone Guard grey/amber, and never resemble active Hook VFX, Ability pickups or Reaction indicators. Palette is a later art pass.
- **Authored Part/Model geometry only**, no Terrain (Hook sweeps, line of sight, camera collision and floor raycasts read the folder roots).
- **Visible geometry and collision agree.** No invisible blockers at lethal edges, decorative walls without collision, oversized proxies, or hanging chains/crystals that can catch a Hook pull.
- **Section seams are visible**, subtle, not grid lines, so the Collapse reads as the world following fracture lines.
- **The greybox must carry:** slab thickness; lethal lip trim; Safe parapets; Risky exposed decks; aprons; Ribs; Dais plinth; Sentry plinths; one seal anchor per Crown Cache; one landmark placeholder per Shard; visible section seams; undersides believable once a section is removed.

#### Arena v1 hazard policy (**fixed**)

- Environmental pressure is only void/lethal edges, Crown Sentries, future Wardens and the future Collapse. **No map hazard deals damage**, and the environment eliminates only through Void.
- No vents, pulses, eruptions, temporary zones, collapsing bridges, disappearing floors or moving platforms or cover. **No placeholder scripts, hazard anchors or generic hazard-marker framework.**
- A vent or eruption may return later under the constraints in [`rift-crown-design.md`](rift-crown-design.md) (Point 11), each with its own ticket and data model referencing `ShardId` and `SectionId`.

#### Required configuration values

Everything below lives in `Config` (or the existing tuning module), not in logic. Names are an implementation decision. All current values are **tunable** starting points unless noted.

| Key | Start |
|---|---|
| `arrivalHoldSeconds` | 3 (0 = off) |
| `crownOpenSeconds` | 60 (45–90), finite, ≥ 0 and below the Match length |
| `voidKillY` | −40, below the lowest underside with a margin |
| `maxPlayers` (Arena v1) | 6 (also the number of pads) |
| `displacementCredit` grounded seconds / hard expiry | 1 continuous / 6 |
| `kitDropInwardOffset` | 6 |
| `pveSlotClearance` | 45 (planning constant for the validator only) |
| Cache kinds `Home`, `Forward`, `Crown` | health 75, 75, 120 and drops 1, 1, 2 (carried over) |
| Pad radius; Home band; Forward rule | 100; 35–50; farther than both Home Caches, Gate-facing, about 50–58 as a tunable target only |
| Section layer radii | per the partition table |
| Spread table | 2→opposite … 6→all, with random rotation |
| Parapet height/thickness, connector widths, gap bands | per the tables above |

The loot bag is the pool times equal copies. 24 drops across 4 Abilities is 6 each, and `Tuning.loot`'s comment ("4 each of 16") must be updated.

#### Build and swap strategy (settled in review, confirmed by the developer)

**Locked:** *Build and structurally validate the new Rift Crown outside the active gameplay query roots first, then perform a reversible swap into the named active folders for combat and runtime validation.* The old and new maps are never active together, because Hook, Tidal, Sentry line of sight, Cache discovery and the rest read only the named roots. The old map stays as a **reference**, not a playable map: after this ticket's code and data changes it is not required to run. The swap is explicit, reversible and minimal, and is not a multi-map framework.

- **Build alongside, never in the live map.** The existing `Workspace.MatchMap` (46 parts at about x 0, z −450) is not touched. The Rift Crown is built as `Workspace.RiftCrownBuild`, centred at about **x +600, z −450**, clear of the old map (±127), the Arena (±52 at the origin) and the Hub (z about 300).
- **Why that is safe.** Every world query reads only the three folders in `Environment.FOLDER_NAMES` (`Arena`, `MatchMap`, `Hub`), and pad, Cache and Sentry discovery read `Workspace.MatchMap`. So a differently named folder causes no raycast ambiguity, duplicate pads or marker conflicts, and no multi-map framework is needed.
- **Consequence.** While it is the build folder, Hook sweeps, Sentry line of sight, camera and floor raycasts ignore it. So before the swap you can run the structural validator (it takes the root as an argument) and the movement measurements that need only physics (walk times, jump, Dash, fall time, pad sight lines, part count). **Anything that needs the world queries (Hook, Tidal against parapets, Sentry sweeps, the seal, a full Match) runs after the swap.**
- **Swap (Edit mode, reversible):** move the old `Workspace.MatchMap` to `ServerStorage.MatchMapGreybox`, rename `RiftCrownBuild` to `MatchMap`, and start a new Play session. Reverse it to go back. No folder-discovery code changes.
- **What the swap does not protect.** The old greybox's markers use the `Outer` and `Central` Cache kinds, which this ticket replaces, so the old map would not run on the new code. Treat `MatchMapGreybox` as a reference. The rollback for code is the last published checkpoint (check the Checkpoints table at the start) and Studio version history, as ADR-0005 says.
- Order: (1) the shell, markers and metadata in the build folder; (2) the structural validator and its specs; (3) the pure Rules and their specs (spread table, roster cap, the Ability deal, credit, hold timing, seal timing, drop offset); (4) the swap and the server integration (hold and GO, seal, kill plane, Cache kinds); (5) the measurements and play checks; (6) telemetry. Each step is testable before the next.
- Logic goes in `Rules` with specs. Anything that reads the world (kill plane, floor test, raycasts) is passed in, as elsewhere. The hold, the seal, the void kill and the credit are server-only, and the client gets presentation events only.
- Use `CLAUDE.md`'s division of work: the main session designs and reviews, `studio-implementer` makes exactly specified edits, `studio-tester` runs long tests. Read `docs/testing/studio-mcp-quirks.md` first and record new quirks there.

#### Assumptions that no longer apply

- Ticket 18's Cache kinds (`Outer`, `Central`), counts (10 and 3), the 16-drop bag and `Tuning.loot`'s "4 each of 16" comment.
- `LootRules.nearestPairs` and its best-effort variety, replaced by the guaranteed triples.
- Ticket 18's and Ticket 20's pacing evidence and hypotheses for the old map (for example the first-Ability and full-Kit times recorded there). The Rift Crown sets a **new baseline**.
- `assignSpawns` wrapping past the pad count, and the spec that tests it.
- Marker names used by `CachesSpec` and `MatchMapSpec` (for example `CacheOuter01`, `CacheCentral2`).
- A fall below −500 as the usual way out of the map (`Removed`, with `lastGrounded` as the drop). With the Rift Crown it is a safety net only.

#### Structural validator (data only, runs in `RunAll` against a root passed in)

Fails the ticket unless **all** hold:
1. Every `CollapseParent` exists and has a strictly later `CollapseLayer`.
2. Every collapsible section has a unique valid `SectionId`, and every Shard section has a valid `ShardId`.
3. Every Cache, Sentry, Warden and spawn marker references an existing section.
4. Every connector has valid `ConnectorType` and Collapse metadata.
5. No required inward parent route depends on a Dash-only route or a Risky connector.
6. Exactly one `CollapseCenter`, six pads, 12 Home + 6 Forward + 3 Crown Caches (24 drops), 3 Sentries.
7. Placement rules: no Cache or drop point within its lip margin, no marker on a connector or apron, no PvE slot within `pveSlotClearance` of any pad, `voidKillY` below the lowest underside, and for every Shard's **Forward Cache**: it is farther from the pad than **both** Home Caches, it lies in the Gate-facing half of its Shard, it satisfies the lip margin, it is on no connector or apron, and it is inside valid Shard geometry. **50–58 studs is not checked as a pass or fail band.**
8. Connector widths and gaps are inside their bands, and no Shard-to-Shard gap exceeds about 30.

It also fails on deliberately broken copies (a missing parent, a duplicate `SectionId`, a Cache on an apron). The Sentry push sweep and the pad sight-line check need raycasts, so they are Edit-mode check scripts run on the build folder or the swapped map, with their results recorded in the ticket.

#### Acceptance checks (automated, structural and spec-level)

These prove rules and data. They do not prove that anything feels fair.

1. The validator passes and fails on its broken copies, and the placement checks and sweeps above pass.
2. **Deal specs** (many seeds): the six invariants hold, a pool of the wrong size is refused, and Shard triples and slot order vary between seeds.
3. **Spawn and roster specs:** the spread table for 2–6 players with rotation; **seven eligible players form a six-player roster and leave one in the Hub, still eligible**; `assignSpawns` refuses more ids than pads without wrapping.
4. **Hold and GO specs:** before `goAt` movement is frozen and `canAct`, collection and Sentry targeting are refused; forged movement and cast requests during the hold have no effect; after GO they work; `startedAt` is set at GO; `arrivalHoldSeconds` = 0 starts at once; leaving during the hold works.
5. **Seal specs:** a sealed Cache ignores every damage source and produces no loot; Sentries are active from GO; all three open at the configured time measured from GO, not from the hold or character creation; `crownOpenSeconds` = 0 opens at GO; invalid values disable Matches with a reason; the Match ending first fires nothing.
6. **Void specs:** the `Void` cause is accepted end to end; kill only for an Alive player in a Running Match after GO; credit rules (Hook, Knockback, expiry, replacement by a newer record, Sentry gives none, self-fall, attacker gone, brief contact does not clear); the drop offset, its validation, fallback and spread-point check.
7. The Reference environment and the Hub are unaffected by `voidKillY`.
8. After the swap, a solo Match and a two-client Match reach Results, `RunAll` and the Ticket 20 regression still pass, and the specs listed in Scope are updated.
9. **Every measurement below has its value recorded in this ticket, and its consequence applied or explicitly kept.** That is the check, not the value.

#### Measurements that must be verified in Studio

Each row gives what we expect, whether there is a pass threshold, and what the result can change. An expected value is **not** a threshold. A row without a threshold only has to be recorded and its consequence applied.

| # | Measurement | Expected (not a pass criterion) | Threshold | It decides |
|---|---|---|---|---|
| 1 | Running and standing jump distance | about 8.7 running (computed, never measured) | none | the jumpable-gap band, and whether the 10–14 Dash gaps are also jumpable (if so, widen them) |
| 2 | Dash distance, and Dash + jump chaining | Dash about 21 | **Dash gaps cross in at least 9 of 10 attempts; the 18–24 bridge-required gaps do not cross reliably.** Otherwise widen those gaps toward 24, and escalate if 24 is still crossed (never past 30) | Dash gap and bridge-required gap widths |
| 3 | Does gravity act on a target during Hook's 0.25 s pull, and does Hook carry a target over each intended gap | unknown | none | whether Hook kills over Dash gaps and Risky bridges work as designed |
| 4 | Smallest obstruction that stops Hook, Hook against a parapet, and from a lower caster | unknown | **a Safe-connector parapet blocks a sideways Hook 10 of 10** | parapet height, the Dais pull behaviour |
| 5 | **Tidal Bolt tap and charged displacement** | the docs disagree (see below) | none | the lethal-edge band, Risky bridge width, every lip margin (Cache 15, platform 10, pad 25–30), the Sentry buttress and the 10-stud rule |
| 6 | Parapet against Tidal (tap and charged), minimum height and thickness | unknown | **a tap does not push a player over a Safe parapet, 10 of 10** | the Safe connector geometry |
| 7 | Melee and Arc Burst reach across +4 and +5 | Basic Attack ±5 and Arc Burst ±8 allowance | none | ruin platform and Dais heights (adjust local geometry first) |
| 8 | Fall time from y = 0 to `voidKillY`, default fall damage, whether a falling player can Dash back | about 0.64 s for 40 studs (computed, never measured) | **`voidKillY` is below the lowest underside by a margin and elimination follows the lip crossing within about 1 s** (proposed, tunable) | `voidKillY`, and whether "no recovery" stays |
| 9 | Walk times: pad → first connector, → Crown, → neighbouring pad, opposite pad | the settled targets (2–3.5 s, about 6 s, 8–9 s, about 17 s) | **within about ±25%** (proposed, tunable) | Shard and pad geometry |
| 10 | Pad-to-pad sight lines and lip margin | none between any two pads, margin 25–30 or more | **no unobstructed pad-to-pad line; margin at least 25** (settled) | pad placement |
| 11 | Sentry push direction and range against each bay | pushes toward the Dais | **no standable point is pushed over a lip** (settled rule) | the buttress and Sentry positions |
| 12 | **Part-count baseline** (see below) | the old map is 46 parts | none, no ceiling | whether to simplify before any polish |

**The Tidal contradiction stays visible.** `docs/testing/studio-mcp-quirks.md` says in one place that a Tidal Bolt pushes its target **25 studs on a tap and 40 on a full Charge**, and in a later note that a tap moves its target **about 5 studs** (25 studs/s for 0.2 s) and that the 20–25 seen earlier included the target's own movement and falls. Older docs also give the charged push as 8 or 40. **No lethal-edge band, Risky bridge width or lip margin may be derived from either figure.** Measure the tap and the charged displacement first, then set those values. If the charged push is far larger than the lip margins can absorb, re-grill whether it is a map or a combat problem.

**Part-count and performance baseline (row 12).** `StreamingEnabled` is false. After the structural greybox is complete (before any visual pass), record: the total `BasePart` count under the Rift Crown, whether client or server frame behaviour visibly regresses in a 2–6 player Studio test, and any construction pattern that repeats parts obviously (for example many identical seam strips that could be one part). Set no ceiling unless the project already has one, and open no optimisation work from it.

#### Studio play checks (judgement, recorded as notes, not unit tests)

These need a human and the real combat. They cannot be proved by specs. Run the first four counts below at 2, 4 and 6 players.

| Question | 2 | 4 | 6 |
|---|---|---|---|
| Is the opening spread readable, with no pad sight line and no instant fight? | ✓ | ✓ | ✓ |
| Is a full Kit reachable on the home Shard in about 25–35 s, with no Rank 2 from it? | ✓ | ✓ | ✓ |
| Does the Crown opening create useful convergence, and does an early rush gain no loot? | ✓ | ✓ | ✓ |
| Does the Arena feel alive rather than dead at this player count? | ✓ | ✓ | ✓ |
| Do Hook and Tidal kills feel fair on Risky bridges and lips, and are they impossible on Safe parapets? | ✓ | | |
| Are Safe, Risky, Dash shortcut, lethal edge and safe edge understandable at a glance? | ✓ | ✓ | |
| Are accidental self-falls (and falls after Dash) rare enough? | ✓ | ✓ | |
| Does the Kit drop after a void death land somewhere reachable without a lip fight? | ✓ | ✓ | |
| Are unused Shards playable and lootable, and is every Shard reachable without Dash? | ✓ | ✓ | |
| Does a roster of seven form a six-player Match and leave the seventh in the Hub? | | | ✓ |

Also walk each Safe, Risky and Dash route once, fight on the terrace and the Dais ramps, retreat with Stone Guard, and use Arc Burst across a Rib.

#### Telemetry and pacing evidence

Add (names are an implementation decision): `CrownOpened match= seconds= alive= nearCrown=` (players within 70 of the centre); `CacheBroken kind=Home|Forward|Crown`; time to a full Kit and to Rank 2 per player; whether the first Crown break came before or after the open; `Void` eliminations with source (Hook, Knockback, Sentry, self), the credit record's age, and `fallsAfterDash`; Match duration and how it ended; the part-count baseline.

**Pacing risk (recorded concern):** before the Collapse exists, the real Arena may permit mid-match stand-offs or inactivity. Do **not** solve it with hazards. Capture: Match duration; time between meaningful interactions; time spent with players in different Shards; time from a full Kit to the first PvP; stretches with no PvP damage, Cache interaction, PvE interaction or rotation; and where survivors are during long idle spans. A dead mid-game becomes evidence for the Collapse or a later hazard ticket.

#### Known future hooks

- **Rift Collapse:** `SectionId`, `CollapseLayer`, `CollapseParent`, `CollapseCenter`, slab undersides, seams, the structural validator, the Dais as an optional final island.
- **PvE:** reserved Warden and elite slots, Sentry plinths, spawn-safety rule, Stone/Rift visual family.
- **Match Intro:** controlled Rift insertion as presentation only, over server-placed positions, with `arrivalHoldSeconds` as the timing contract.
- **Larger maps:** free-drop reconsidered only for materially larger Arenas and populations.

#### Deferred and unresolved

- ~~Rift Collapse mechanics, timing, telegraph, damage, and whether it ends at the Inner Crown or the Dais.~~ Resolved by [Ticket 22](#22-rift-collapse).
- ~~Fate of **Crown Sentries** (Layer 3) and of **loose Pickups and Kit drops** on a collapsing section.~~ Resolved by Ticket 22 (Sentries are removed with Layer 3, loose loot is consumed, Kit drops use a live-floor fallback).
- Warden stats, AI and aggro range, and the Warden slot hypothesis.
- XP, equipment/Relic slot, loot beyond the 24 drops, Rank pacing tuning.
- Cache stagger after the seal, the Match Intro, final art, audio and palette. (The Collapse's art, audio and palette are staged in [Ticket 26](#26-rift-collapse-visuals-and-atmosphere), added 2026-11-10.)
- Kill attribution for any future hazard or PvE displacement.
- Whether the six-player cap is permanent, and a Dash ledge guard (only on telemetry evidence).
- Whether a dead mid-game justifies any hazard.
- Which six players are kept when more are eligible (random for now, "longest waiting" later if Hub order is ever recorded).

#### Integration contract for Ticket 25 (added 2026-11-10)

[Ticket 25](#25-match-milestones) (Milestones, **Stage 2**) reads from this build. These are requirements, with no change to the layout:

- Each player's **start Shard** (from the pad `Match` assigns) is available to server code after setup.
- A **Shard-membership lookup** from a position or a `SectionId`, using the `ShardId` metadata this ticket already adds.
- (Stage 3 only) the Kit-drop spread points and their validation must cope with **up to three** dropped items, not two.

#### Presentation metadata for Ticket 26 (added 2026-11-10)

[Ticket 26](#26-rift-collapse-visuals-and-atmosphere) (Collapse presentation, **specified, not built**) needs a little authored presentation metadata so that warning effects match the real geometry. This amends the "and nothing more" in the Rift Collapse non-goal above: it adds **presentation metadata only**, with **no gameplay scope, no runtime geometry system and no change to the layout**. It is built during this ticket or as the first step of Ticket 26's Stage 1.

- Each Section and connector carries an ordered **`CutLine`** set of Attachments along its inward breaking boundary, with the fewest points that represent it accurately.
- Each Section has a **`VfxGroup`** attribute, defaulting to `ShardId` unless the authored layout needs different grouping. Rim links and connectors are assigned during review.
- One **`BeaconAnchor`** per `VfxGroup` **per `CollapseLayer`**, because a Shard spans several layers.
- Anchors are **generated once at authoring time** from the authored geometry and reviewed for irregular boundaries.
- The structural validator gains **additive** checks: the required anchors exist and `CutLine` points lie on or near the Section's boundary. A missing required anchor fails validation before release.
- A missing anchor at runtime never disables a warning. Ticket 26 logs it and omits a cut-line it cannot draw accurately.
- Attachments are not geometry. They don't affect collision or queries and travel with their Section model.

#### Dependencies, risks and ADRs

- Depends on Tickets 15–20 (Kit, Pickups, Match flow, Caches and Sentries, Hook, run-test readiness). After the swap the Ticket 20 regression must still pass.
- **Part count:** measure it as above, and simplify before polish, not after.
- **Greybox first.** Resizing at the greybox stage is the intended way to fix a wrong number.
- **No ADR required.** The existing ADRs record durable architecture (authority, movement, the status model, hitboxes, the MCP workflow, charging, one life). `voidKillY`, the kill-plane model and the 24-drop economy are tunable gameplay design, recorded here and in the design archive. Revisit only if one of them becomes hard to reverse.
- The glossary terms for this map were reviewed against this ticket and carry no tunable numbers.

#### Build record (2026-11-11): steps 1 to 4, not complete

Built in the order the Build and swap strategy sets. Nothing was published or uploaded. `Tuning.kit.startingKit` is `"Reference"`, and no TEMPORARY aid remains.

**State of the place.** `Workspace.MatchMap` is the Rift Crown (swapped in), and the old greybox is `ServerStorage.MatchMapGreybox`, as a reference only. `RunAll` on the Server in a fresh solo Play: **498 passed, 0 failed** (427 at version 98). With the Reference Kit there is no `MatchPhase`, `MatchGoAt`, `Caches` or `Sentries`, and the player spawns in the arena as before.

**Step 1, the shell.** The map is generated by `ServerStorage.Tools.RiftCrownGreybox` (an Edit-mode tool that nothing in the game requires): `require(...:Clone()).build()` rebuilds `Workspace.RiftCrownBuild` from one table of dimensions, so a resize is a number change and a rebuild. It never touches `Workspace.MatchMap`, so after a rebuild the swap is done again by hand.

- **Shape.** Six wedge Shards on a constant **24-stud seam**, from r 48 to r 118, then three rim teeth of different lengths (to r 130–133). A Shard is about 28 wide at its inner edge and 108 at the fan's end, about 6,000 studs² before its ruins. Centre at (600, 0, −450).
- **Sections (15):** `Shard<n>Outer` (Layer 1, r 85 out), `Shard<n>Inner` (Layer 2, r 48–85), `CrownOuter` (Layer 3, r 24–32), `CrownInner` (Layer 4, with the ramps, Ribs and seal anchors) and `CrownDais` (Layer 5, a pillar from y −24 to +5). Slabs are 14 thick with a keel to y −22. The lowest underside is −24, so `voidKillY` −40 has 16 studs under it.
- **Connectors (12):** six spokes (`Spoke1`, `3`, `5` Safe at 16 wide with parapets, `2`, `4`, `6` Risky at 9 wide), three Safe pair links (`PairLink12`, `34`, `56`, 18 wide, at s 59–77 along the seam), the Risky rim link `RimLink45` (9 wide, at s 101–110), and two Dash shortcuts (`DashGap23`, `DashGap61`), which are marker Parts between two aprons that pinch the seam to 12.
- **Markers (29) and pads (6):** in `Markers` and `Pads`. Pads `MapSpawn1`–`6` at (103, 0) in each Shard's frame, invisible, facing the Crown, each behind a three-wall ruin. `CacheHome<n>A`/`B` at (114, ±33.5), `CacheForward<n>` at (65, 4.5 toward the Gate), `CacheCrown1`–`3` at r 19, `Sentry1`–`3` at r 28 on 2.5-high plinths, `WardenSlot1`–`3` at (57.5, 9 toward the Gate) on the Safe-spoke Shards, `EliteSlot` on the Dais and one `CollapseCenter`.
- **Attributes.** Sections: `SectionId`, `CollapseLayer`, `CollapseParent`, `ShardId`, `Adjacent` (the Sections it touches as solid ground), `ShardAzimuth`, `GateAzimuth`. Connectors: `ConnectorType` (Safe, Risky, Dash), `ConnectorRole` (Spoke, PairLink, RimLink, DashShortcut), `CollapseLayer`, `FromSection`, `ToSection`, `Width`, `Gap`. Markers and pads: `SectionId`, `ShardId`, and `CacheKind` or `SlotKind`. Apron and landing paving carries `Apron`.
- **Part count baseline (row 12): 322 BaseParts, 171 of them colliding** (the old map had 46). The 151 that never collide are lip trim, seams, paving and markers. The obvious repeat is the lip trim: about 80 thin strips, one per edge. No ceiling set and no optimisation opened.

**Decisions taken while building, for review:**

| Decision | Why |
|---|---|
| The Section split between Layers 3 and 4 stays at r 24. Ribs are at r 22.35–23.85 and each Dais ramp runs from r 13.4 to 22.3 (about 29°), so a ramp's foot touches its Rib and is entered from the sides | Ticket 22 needs the ramps and Ribs in Layer 4. The Dais is also jumpable (5 studs against a 7.2-stud jump) |
| Each Sentry stands in a buttress **niche**: a back wall and two cheek walls, 10 high, above its firing point | With only a wall behind it, every Sentry saw the spokes, the landings and the other bays. In the niche it sees a cone of about ±34° into its own bay, which covers every breaking spot and drop point |
| Pads are on the centre azimuth at r 103, not shifted | It gives the measured walk times below. A pad shifted toward one seam put neighbours 4 s apart |
| Home Caches sit at the two outer corners of each Shard, 35.3 from the pad | The only places in Layer 1 that are both 35 from the pad and 15 from a lip |
| The Forward Cache is 38.3 from its pad, not 50–58 | The inner throat is only 46 wide at r 65. The structural rule holds: farther than both Home Caches, on the Gate's side, 15 from a lip |
| The Warden slot is 46.4 from its own pad and 8.7 from the Forward Cache | `pveSlotClearance` 45 leaves only the throat. Ticket 23 refines the slot |
| The Crown's opening is published as attributes (`MatchCrownOpensAt` on Workspace, `CacheSealed` and `CacheOpenedAt` on each Crown Cache), with no new remote | The existing world state is attributes, and a late joiner reads the same state |
| The seam is 24 everywhere, the top of the bridge-required band | Row 2 expects Dash plus jump to cross less, and 24 is where the ticket says to start widening |

**Step 2, the structural validator.** `MapRules.validate` (pure, eight numbered rules) on the plain layout that `MapLayout.read` makes from the folder, with the world questions (floor owner, lip margin by a ring of floor probes, aprons, lowest underside) answered by raycasts against that folder alone. **0 problems on the map as built.** `MapRulesSpec` breaks a written-out layout one rule at a time, and `MatchMapSpec` runs the validator on the real folder, then on copies with a missing parent, a repeated `SectionId` and a Cache probed on a real apron. `Match` refuses to run Matches on a map that fails it. `MapLayout.sectionAt`, `shardAt` and `Match.startShardOf` are Ticket 25's lookups.

**Step 3, the pure Rules.**

- **`LootRules.deal`** is the guaranteed deal. It holds the five rules of `LootRules.check` over 2,000 seeds for the real pool, and for pools of 3, 6, 8, 12 and 24. Pools of 2 and 5 are refused with a reason. `nearestPairs` and the variety repair are gone.
- **`MatchRules`:** the roster cap (`buildRoster` with `maxPlayers` and the injected Random), the spread table (`Tuning.map.spread`) with a random turn and a random hand-out, `assignSpawns` refusing instead of wrapping, `arrive` and `hasGone` for GO, the gates taking `now`, `inVoid`, and `addKill`.
- **`VoidRules`:** the displacement credit (newest wins, a second of unbroken floor or six seconds ends it) and the inward Kit drop with its per-item check.
- **`CacheRules`:** `isSealed` and `validateSeal`.
- **The order of draws from the Match's shared Random is now fixed:** the deal, then any roster cut, then the spawns.

**Step 4, the swap and the server.**

- **Arrival hold and GO.** `Match` places everyone, anchors each root on the server, and calls `arrive`. GO re-bases `startedAt` and the deadline, and tells `Session`, `Caches` and `Sentries`. Before GO nobody can act, collect or be hit, so no Sentry targets anyone and no Cache takes damage. A Match that ends inside the hold lets its survivors go.
- **Crown seal.** A Crown Cache is sealed from the moment it is built. `Caches.onHit` asks the clock before anything else, and `Caches.update` opens each one once. `Match` logs `CrownOpened match= seconds= alive= nearCrown=`. An unusable `crownOpenSeconds` disables Matches at startup.
- **Void.** The Match loop eliminates anyone below `voidKillY` after GO, with the cause `Void`. `Combat` reports every push or pull of a player, and `Match` keeps the credit. The killer's kill counts even if they are Eliminated or gone. The body goes after `voidBodySeconds` (1).
- **Telemetry added:** `Eliminated ... void=Hook|Knockback|Sentry|self creditAge= afterDash=`, `Kill ... void=yes`, `CacheBroken kind=Home|Forward|Crown` with `sinceOpen=` on a Crown Cache, `LootDeal ... crown=`, and `CrownOpened`.
- **Clients:** `Input` passes nothing before GO, `MatchHud` shows "Get ready — 3" and the Void's wording in the feed, the title and the results, and `PveVisuals` draws a placeholder band on a sealed Cache and a burst with a low tone when it opens. Placeholder built-in sound only.

**Measured so far (solo Play, a server-driven walker at real WalkSpeed):**

| # | Measurement | Result | Against |
|---|---|---|---|
| 8 | Lip to `Void` elimination | **0.82 s** | threshold about 1 s: passes |
| 9 | Pad → first connector | **3.0 s** to the Dash apron, 3.6 s to the Safe pair link | 2–3.5 s |
| 9 | Pad → spoke start → Crown rim | 4.7 s, then **5.8 s** | about 6 s |
| 9 | Pad → neighbouring pad over the Safe pair link | **8.7 s** | 8–9 s |
| 9 | Pad → opposite pad | not walked: 15.6–16.5 s from the measured legs plus a 64–78 stud crossing | about 17 s, within 25% |
| 10 | Pad sight lines and lip margin | no clear line between any two pads (a spec), margin 25 by the validator | passes |
| 11 | Sentry push at its bay's spots | none leaves the floor at twice the tap push (a spec) | passes |
| 12 | Part count | 322 | recorded |

**Findings for the developer, none fixed by a design change:**

1. **All three Sentries see a player standing on the Dais.** Their firing points (y 8.1) are above the Dais top (5), and the niche's cone includes the centre. On the terrace each bay is seen by its own Sentry only. Until Layer 3 falls, the Dais is a place where three Sentries fire at once.
2. **The Crown's outer band is not a ring.** Each Sentry niche closes it at its bay (0.7 studs are left at the rim), so going round the Crown means passing in front of a Sentry at r under 25, or crossing the terrace or the Dais.
3. **A Sentry's tap pushes a player near the rim outward** wherever it can see them. The niches now hide the band, the landings and the spokes, and a spec checks that. The bay's own spots are pushed inward.
4. **Dash plus jump is expected to cross the 24-stud seams** (20 for the Dash plus about 8.7 for a running jump). Row 2 has not been measured. If it does, the ticket says to escalate, never past 30.
5. **The Tidal config says 5 studs on a tap and 8 on a full Charge** (25 and 40 studs/s for 0.2 s). Row 5 has not been measured in Play, and no margin was derived from it.
6. `ServerStorage` holds eight stray `PickupsSpecFolder` folders from before this ticket, probably from `RunAll` in Edit mode. Left alone.

**Still open before the ticket is done:**

- Rows 1 to 7 of the measurements (jump, Dash and Dash plus jump, Hook over gaps and against a parapet, Tidal tap and charged, parapet against Tidal, melee and Arc Burst reach across +4 and +5), which need real input or forged Casts from a client, and a walked row 9 for the opposite pad.
- A solo and a two-client Match played through to Results with real combat, a Crown Cache broken after the open, a real Hook or Tidal kill into the Void with its credit, and a Kit drop after a Void death.
- The Studio play checks table at 2, 4 and 6 players, and seven in the Hub.
- The developer's review of the decisions and findings above.

**Settled with the developer (2026-11-11):** finding 1 is kept (the Dais stays under all three Sentries until Layer 3 falls, and the play checks decide whether it needs softening). Finding 2 is accepted (the outer band stays closed at each bay). Finding 4 waits for row 2. Measurements start in solo Play, and the two-client Match and play checks wait for a Server & Clients session.

**Rows 1 and 2, measured 2026-11-11 (solo Play, `studio-tester`).** Rendering died mid-session, so only 2 of the 5 running jumps used real input. The rest drove the client's real `ForcedMovement` and `Humanoid` from a 60 Hz Heartbeat trigger, which is best-case timing.

| Row | Result |
|---|---|
| 1 running jump | 8.0 studs (2 real runs; emulated 7.1–7.8, understated by slow acceleration). Standing jump 0 horizontal, peak about 7.2. Airtime 0.5 s |
| 2 Dash, flat | 19.7–21.1, mean 20.7 (0.25 s at 80 studs/s) |
| 2 Dash across the 12-stud `DashGap61` | 10/10 when the Dash starts in the last 8 studs before the edge. From 9.5+ studs out it falls (3 of 5). A Dash is level and drops about 6 studs in 0.25 s, so the window is short |
| 2 jump then one Dash across a 24 seam | 7/10 crossed, reach 22.7–25.1 (Dash within 0.3 s of the jump) |
| 2 Dash then jump | 2/10 |
| 2 jump + two Dashes | **6/6 crossed, reach 26.6–40.0** |
| 2 Dash, jump, Dash | **4/4 crossed, reach 34.6–42.1** |
| 2 two Dashes, no jump | 0/3 |

**Finding 4 is confirmed and escalated:** a 24-stud seam is crossed by one Dash plus a jump with tight timing, and by two Dashes plus a jump every time, past the 30-stud "never" line, so widening cannot stop it. **Settled with the developer (2026-11-11): accepted as skill play.** The seams stay at 24, Dash is unchanged, and the play checks say whether it breaks routes or the Collapse's parent rule. `DashGap23` was not walked itself (same 12-stud pinch as `DashGap61`).

**Rows 3 to 7, measured 2026-11-11 (solo Play, `studio-tester`).** The victim was the real player character, so the client's real `ForcedMovement` ran. The caster was an anchored dummy. Charged Tidal Casts replicated `Casts.deliver` with the real scaled stats (not the forged ChargeStart/ChargeRelease path).

| Row | Result |
|---|---|
| 5 Tidal tap | 5.07–5.60 studs, mean 5.4, over 0.22 s, peak speed 25 |
| 5 Tidal full Charge | 8.37–9.04 studs, mean 8.65, peak speed 40.5 |
| 5 which figure is true | **The Config's 5 and 8 studs.** The "25 and 40" in older docs are speeds in studs per second. The quirks file's contradiction is corrected. No lip margin needs more than about 9 studs for a Tidal push |
| 6 parapet vs Tidal, grounded victim | tap 10/10 held, charged 5/5 held (**passes the threshold**). Parapet 1.5 thick × 4 high, 6.5–8.0 from the deck centre line |
| 6 parapet vs Tidal, **jumping victim** | **2/2 went over and fell into the void** (not part of the threshold) |
| 4 Hook vs parapet, sideways, same-level caster | 10/10 blocked (**passes**), but the margin is **0.1 stud**: the projectile leaves at y 3.9, the parapet top is y 4.0 |
| 4 Hook, elevated caster (+2 to +6) | the projectile passes over every time. The pull is then stopped by the server sweep, 2.0 studs short of the face, or by physics if the victim touches the wall |
| 4 smallest obstruction | LowCover (3.5 high) blocks the projectile only from 4.5+ studs; closer, it skims over. A pull is stopped by anything about 1.7+ high. Rib (8.1) assumed to block, plinth not isolated |
| 4 Hook from the terrace to a Dais target (+5) | hit, pulled about 10.5 studs, then fell the 5 studs to the terrace |
| 3 gravity during the pull | **Yes.** The pull sets level velocity only |
| 3 Hook over `DashGap61` (12 studs) | pull speed is distance / 0.25 s: a 15-stud pull (60 studs/s) fell into the void, a 19-stud pull (76 studs/s) carried over. Threshold about 65–70 studs/s, two casts only. A Hook with the caster close to the far edge is a kill over a Dash gap |
| 7 Basic Attack reach | 9.5 studs at dy 0, +3.5 and +5 (range 8 plus 1.5 capsule), no loss on the Dais or a ruin platform |
| 7 Arc Burst reach | tap 19.5 at dy 0, +3.5 and +5. Full Charge 29.5 flat only |

**Settled with the developer (2026-11-11): the jumping-victim crossing and the thin Hook margin are accepted for now.** The parapets stay 4 high, and the play checks judge whether a jump-then-hit kill on a Safe connector feels unfair.

**Two-client Match, 2026-11-11 (Server & Clients, `studio-tester`, `startingKit` set to `"Empty"` for the session and reverted to `"Reference"` after).** Five Matches ran. Both clients rendered at 15 fps, so casts were forged and movement server-driven.

| Check | Result |
|---|---|
| Two players on opposite Shards | pass in all 5 (pads 5/2, then 3/6 three times; the repeat is chance, as `Match` uses an unseeded `Random.new()`) |
| Arrival hold | pass: `goAt` = Running + 3.000 s, positions constant until then. Forged casts and a Cache attack from 5 studs did nothing, but the Cache refusal wasn't independently proven by range (the teleport used a TEMPORARY helper) |
| 24 drops, 21 Caches, 3 Sentries | pass. Every Shard's three drops were three different Abilities, Crown Caches held two each |
| Crown seal | pass: 14 forged hits at ~27 s left a Crown Cache sealed, `CacheStage` 0, no drop. `CrownOpened seconds=60.2`. After the open it broke (`CacheBroken kind=Crown sinceOpen=23.1`, two Abilities dropped) |
| Sentries active | seen killing a player who stayed in a bay (not observed before GO) |
| Void kill, Tidal tap | pass x2: `Eliminated cause=Void void=Knockback creditAge=1.0/1.1`, `Kill ... void=yes`, killer credited, eliminated at y −57 about 1 s after the hit |
| Kit drop after a Void death | pass: the Pickup landed 6 studs inward on valid floor |
| Self-fall | pass: `cause=Void void=self`, no killer |
| Results, return to Hub, Results wording | pass: "Eliminated by Player1", "Fell into the Rift", "Winner" |
| Rift Hook as the credit source, the Void feed line | not tested |

**One unexplained result:** in Match 1 one fall at the same lip spot gave `cause=Died by=none` (log: `Death cause=NotCombat`, `Fight Lost taken=12`, then `Eliminated cause=Died` 2.5 s later), where three later falls gave `Void`. No trace was running. The leading guess is that the victim was already low on health and died of the tap before the Match loop saw the void, but the missing killer credit is unexplained, so it stays open. If it appears again, check what `Humanoid.Health` the victim had.

**Follow-up in the same session (`studio-tester`).** Low health is **not** the cause of the Match 1 result. A victim held at 5 or 12 health died of the tap by health and was correctly credited (`Eliminated cause=Died by=player:-1`). One held at 10 regenerated to 12.4, survived and was pushed over the lip as `Void void=Knockback creditAge=0.9`. Full health gave `Void creditAge=1.3`. Every Eliminated line came 1 to 1.3 s after the hit, so the `Died by=none` 2.5 s after the hit has not recurred and **stays open**. Untested guesses: the victim stuck on a rim tooth and died later, or something else damaged it. If it appears again, log the victim's health and y from the hit.

**Rift Hook as the Void credit source: pass.** A 13-stud pull over `DashGap61` (caster 2 studs past the far edge, victim in the gap) gave `Eliminated cause=Void void=Hook creditAge=1.0` and `Kill ... ability=RiftHook void=yes`. A 19-stud pull (caster 10 past) carried the victim over, landing with 8 damage. The `Eliminated` line shows `hookPulledAgo=none` on the Hook kill although `HookPull` was logged 1 s earlier. It may be intended for `Void`, and is not checked. A tap fired through a solid part (the Landmark Stumps at x 600 and 604 on Shard 2) hits nothing and logs nothing, so raycast the line first.

Not done: Risky-spoke edge Hook, Arc Burst full Charge at +3.5 and +5, `PairLink12` (same parapet size as `Spoke1`), a real forged ChargeStart/ChargeRelease hold.

### 22. Rift Collapse

**Status: specified 2026-11-09, built and accepted 2026-11-11 (699 specs, 0 failed; steps 1 to 10, latest code commits `df5a3c7` step 9a, `e3aea46` step 9b and `8e4830a` the boundary correction). The `.rbxl` snapshot is not re-saved and no checkpoint is published (both the developer's).** It was reviewed adversarially the day it was specified. Every design point was grilled with the developer and settled. The reasoning, rejected options and edge cases are in [`rift-collapse-design.md`](rift-collapse-design.md); this entry is the build contract. Every number carries one of four labels:

- **settled initial value:** a design decision that has a number. Changing it is a design change and is recorded here.
- **tunable:** a starting value in `Config` that can change from evidence without a redesign.
- **measurement-dependent:** confirmed or changed after a Studio measurement, with the consequence named beside it.
- **test-only:** exists only in the TEMPORARY accelerated test configuration and is never committed.

**Expected values and balance targets are not pass criteria.** A target (for example a typical Dais fight length) produces a tuning note. It does not keep this ticket open. Only the **Blocking completion criteria** below decide completion.

#### Summary

The Rift Collapse is the time-driven phase that removes the Rift Crown's ground, layer by layer from the outside in, until the survivors fight on the Dais. A fixed schedule is built at GO. Each layer gets a 30 s **Warning**, during which it stays completely normal, and is then **consumed** in one authoritative step. The Collapse deals no damage. Anyone left without ground falls, and the existing `Void` system handles the death. The Collapse is meant to be the normal way a Match ends, and the Match Timeout stays a technical failsafe.

Desired rhythm: **Gather → Clash → Collapse → Final Fight.** These are derived labels used by telemetry, not Match states. The lifecycle stays Countdown → Running → Results, and the Collapse runs inside Running.

#### Scope

- `CollapseRules` (pure) and a small `Collapse` server module that `Match` drives: the schedule, its validation, the ordered event cursor and the effective-schedule shift.
- Section states, Warning, atomic consumption by reparenting, restore, and the intact-map assertion.
- Object cleanup on a consumed Section, the Kit-drop live-floor fallback, and the Sentry cancel.
- The settle window and the Match-end integration, including `maxSeconds` becoming 600 and re-based on `goAt`.
- The Dais Last Stand, with Layer 4 optional by configuration.
- The replicated Match and Section state, the HUD line, the local alert, the world-telegraph states and their cleanup, and the accessibility requirements.
- Telemetry, configuration, validators, specs, structural checks, the Studio matrices, the latency measurements and the Tidal publish gate.
- Updates to `CONTEXT.md` and `docs/testing/studio-mcp-quirks.md`.

#### Non-goals

- **Any Rift damage:** ticks, ramps, storm damage, a damaging, slowing, blinding or pushing volume, Shield or Stone Guard interaction, or a Collapse killer or source for attribution.
- **Any grace after removal:** a floating platform, temporary collision, a rescue timer, a teleport, a recovery state, or immunity for Stunned, Rooted or fighting players. A short **Unstable** stage is the named fallback if evidence shows deaths feel cheap. It is not built here.
- **Layer 5 removal and any overtime:** consuming or shrinking the Dais, anti-stall pulses, sudden-death or combat modifiers, forced-draw mechanics.
- **Dynamic timing:** roster scaling, survivor-based pause or acceleration, per-Section stagger.
- **Loot relocation** (other than the Kit-drop fallback below), a Dais Cache or any new Dais objective.
- **Wardens** beyond the generic despawn rule and the `removeSection` hook. No Warden AI, XP, drops, credit or aggro ([Ticket 23](#23-rift-warden) owns those).
- **Combat changes.** No Tidal, Hook, Stone Guard or Arc Burst retuning. If the Tidal gate fails, Layer 4 ships disabled and Tidal is not changed here.
- Phase labels, a Crown-opening countdown or a second clock on the HUD.
- Final art, audio, VFX, palette, fonts and icons. **No upload without explicit approval.**
- Multi-session or multi-Arena support, pristine-map cloning, a generic world-effect ownership framework, an analytics backend, positional sampling.

#### Dependencies on Ticket 21

**Relies on:** the Section and connector metadata (`SectionId`, `CollapseLayer`, `CollapseParent`, `ShardId`, `ConnectorType`), `CollapseCenter`, markers that carry `SectionId`, the structural validator, slabs 12+ studs thick with sheer faces, the Dais as a self-supporting pillar, `arrivalHoldSeconds` and `goAt`, the Crown seal, `Void` at `voidKillY` with displacement credit and the inward Kit-drop rule, the 24-drop loot economy, Crown Sentries at r ≈ 27 (Layer 3) and Crown Caches at r ≈ 19 (Layer 4).

**Requires from the Ticket 21 build:** the Crown ramps and Ribs belong to Layer 4, and the Sentry plinths and buttresses belong to Layer 3 (the Dais isolation check enforces the first). Each scheduled Section and connector model has exactly one stable home parent, whatever the folder layout is.

**Amends Ticket 21:**
1. `Tuning.match.maxSeconds` becomes **600** (it was 480), and the failsafe deadline is `goAt + maxSeconds`. Ticket 21 already moves the Match clock to GO.
2. The Kit drop after a `Void` death validates against **live** floor with a `CollapseParent` fallback. Where the floor still exists, behaviour is exactly Ticket 21's.
3. The structural validator gains the Collapse checks below, additively.
4. Every item in Ticket 21's "Deferred and unresolved" list that mentions the Collapse is resolved here.
5. Ticket 21's lip-margin rule applies to the **intact** map. After Layer 3 is consumed, Crown Caches (r ≈ 19) sit about 5 studs inside the new lethal lip at r ≈ 24. That is accepted and measured (see Review notes).

#### Collapse gameplay model (**fixed**)

- A Section has two gameplay phases: **Normal / Warning** and **Removed**. During Warning it stays fully solid, traversable, queryable and fightable, with no damage, movement penalty, vision penalty, push or status.
- At removal time `T` the Section is removed from gameplay in **one authoritative step**.
- **Players fight normally on a Warning Section until `T`.** Hook, Tidal Bolt, Arc Burst, Basic Attack, Stone Guard, Dash, Pickup collection, Cache damage and PvE all work. Nothing is pre-disabled because its floor is in Warning.
- **The Rift is the absence of ground.** There is no airborne Rift volume.
- **Death model:** the Collapse removes floor and the existing `Void` system handles the result. The death cause stays `Void`, and recent valid Hook or Knockback attribution still credits the pusher. Otherwise it is a self-fall. The Collapse never becomes a killer or a credit source.
- **Accepted consequence:** a player who is Stunned, Rooted, Frozen or committed to a fight when `T` arrives falls with the Section. The fairness burden belongs to the Warning, the telegraph and a predictable order.
- **Removed geometry invariant (fixed):** after a Section is Removed, **no authoritative gameplay query may behave as though its geometry still exists.** This covers character collision, Hook and `Cover:sweep`, line of sight, Sentry targeting and cover, projectile and environment queries, floor raycasts, Pickup floor validation and the camera and aim cover checks.

#### Section states

| State | Meaning | Allowed next state |
|---|---|---|
| `Normal` | Intact and not warned | `Warning` |
| `Warning` | Intact, telegraphed, `CollapseRemovesAt` set | `Removed`, or `Normal` only through `stop()` at Match end |
| `Removed` | In the holding folder, absent from gameplay | `Normal` only through a restore |

- The authoritative state lives in the server registry. The `CollapseState` attribute on a Section model is a presentation replica for clients, and the server never reads it back.
- **Only `Normal → Warning` and `Warning → Removed` happen at runtime.** There is no `Normal → Removed`, no second Warning and no second removal.
- Connectors are Sections for state and telegraph purposes, with the same states, timestamps and effects. Dash gaps are markers with no geometry and need nothing.

#### Atomic removal and the authoritative boundary (**fixed**)

- **Atomic layer removal.** Every Section scheduled for an event is removed in one non-yielding update. No gameplay code may observe some Sections of the layer as Removed while others sharing the same `CollapseRemovesAt` are not, and there is no `ShardId` or `SectionId` priority. Visual crumble may stagger after the authoritative step. Gameplay never staggers.
- **Boundary invariant.** Before any gameplay handler relies on Collapse-sensitive world state, every scheduled removal whose time has passed must already have been applied. Gameplay before the update for `T` sees the old world, and gameplay after it sees the fully removed world. There is no rollback, no retroactive hit correction, no partial Section state and no replay.
- **Mechanism.**
  - A lightweight Heartbeat step runs while the Match is active (GO to `stop()`).
  - The existing Match-installed gates (`canAct`, `canCollect`, the target filter) and the Sentry loop also call `sync(now)`, so no ability gets its own call.
  - The Heartbeat step keeps the worst-case error of pure geometry queries that don't pass a gate (projectile and Hook sweeps) to about one server frame. If testing shows that is not acceptable, fix it at the shared environment-query layer, not per ability.
- **Consequences at the boundary.**
  - A combat event accepted before the boundary resolves under the old geometry. One accepted at or after it resolves under the new geometry. That covers damage, Hook, projectile sweeps, Sentry fire, Pickup and Cache interaction.
  - **Pickup collection:** removal wins if the collection attempt happens after the Section is Removed. The existing collection cadence (about 10 Hz) means up to about 0.1 s of granularity, which is documented and tested. No bespoke high-frequency loop.
  - **Cache break:** a break completing in the removal tick produces no loot.
  - **Sentry:** a shot may begin or resolve only if accepted before its Sentry is removed.
  - **Projectiles** in flight survive and resolve against whichever world exists at their sweep.
  - **Hook:** no special case. Normal Hook movement plus the removed geometry decide, and displacement credit applies as usual.
- **Safety is the physical outcome.** The server does not classify a player as safe or doomed at `T`, and does not snapshot positions to pre-declare deaths. After removal, a player with live support keeps playing and one without falls. A `Void` elimination occurs only when the existing authoritative Void condition is reached. The server's view of "supported" is its observation of validated client movement (ADR-0002), so the plane crossing is the authoritative record of a fall. Client replication latency may cause a **presentation** discrepancy but never changes who survives, and no hidden server grace compensates for it.

#### Schedule and timing

**Representation (one source of truth):** each row is `{ layer, removeAt, warnSeconds? }`, in seconds from GO, with a global default `warnSeconds`. **`warnAt = removeAt − warnSeconds`** is always derived. Config never stores both. Rows must form a **contiguous prefix of Layers 1–4**.

| Event | Time from GO | Label |
|---|---|---|
| Crown opens (Ticket 21, independent) | 1:00 | tunable |
| Layer 1 Warning / removed | 3:00 / 3:30 | tunable |
| Layer 2 Warning / removed | 4:15 / 4:45 | tunable |
| Layer 3 Warning / removed (**Final Fight begins**) | 5:30 / 6:00 | tunable |
| Layer 4 Warning / removed (**Last Stand begins**) | 6:45 / 7:15 | tunable |
| Match failsafe (`maxSeconds`) | 10:00 | settled initial value |

- Removal offsets in config are 210, 285, 360 and 435 s, **tunable**. `warnSeconds` defaults to 30 s, **tunable**.
- The cadence is 30 s Warning, removal, 45 s stable gap, next Warning, a 75 s rhythm. The first-Warning tuning window is about 2:30–4:00, **tunable** (guidance, not a validation bound).
- The Warning must be long enough for ordinary walking to evacuate a layer (the figure used when the Warning was settled is about 3–8 s of walking per layer), plus time to notice and finish a fight. The intent is that a player who notices the Warning in reasonable time can leave without a Dash.
- **Absolute timestamps.** At GO every offset is converted to an absolute server time **once** (`goAt + offset`), and those are used for the rest of the Match. The Crown seal, the Collapse and the failsafe all use `goAt` as the origin. Timestamps are not recomputed from elapsed time.
- **No dynamic timing.** The schedule never pauses, never accelerates, and doesn't change with survivor count or roster size. 2–6 players share it. If telemetry shows repeatable dead time in 2–3 player Matches, the first allowed experiment is a roster-band shift of the **first Warning only**, with the cadence unchanged. Not built here.
- **One survivor** ends the Match normally after the existing end grace and never waits for the Collapse.
- **Derived labels** (telemetry only): Gather is GO to the Crown opening, Clash is the Crown opening to the Layer 1 Warning, Collapse is the Layer 1 Warning to the Layer 3 removal, and Final Fight is after the Layer 3 removal. A Last Stand sub-label (`dais`) starts at the Layer 4 removal.

**Scheduled, effective and applied times (stall handling).**

- **Scheduled:** the configured offset from GO. **Effective:** the authoritative time after any stall-induced shift. **Applied:** when the update actually ran.
- Events apply **strictly in order**. A later event never applies before an earlier one.
- When an event applies late, `delay = applied − effective`. **That event's successors are all shifted by exactly `delay`**, so the 30 s Warning and 45 s gap are preserved. A late Warning therefore shifts its own removal by `delay` and every later Warning and removal by the same amount. A Warning is never skipped or shortened, and a Warning and its removal are never applied in the same update.
- The HUD and clients use the **effective** times. Telemetry records scheduled, applied, effective and `delay`.
- The failsafe margin is validated against the **effective** last event. If accumulated delay threatens the margin, record it as an abnormal server-performance condition. Do not shorten any Warning.

#### Crown interaction

- `crownOpenSeconds` stays independent of the Collapse and is not derived from Layer 1.
- **Guard:** the first Warning must begin at least `firstWarnGuardSeconds` (**tunable**, 90 s) after the Crown opens. With the defaults that leaves 120 s of Crown contest before any pressure. An invalid combination disables Matches with a clear reason.
- The Crown opening is an **event message** (the existing world and audio event plus a short message). It does not get a countdown in the main status slot.

#### Final Crown and the Dais

| Stage | Time | Playable ground |
|---|---|---|
| Crown Final Fight | 6:00 → 7:15 | Inner Crown (Layer 4) and Dais |
| Dais Last Stand | from 7:15 | Dais (Layer 5) only |

- **Layer 5 is never consumed.** The Dais stands until one survivor remains or the failsafe is reached. There is no further scheduled event.
- **Layer 4 is an optional row.** Omitting it gives the permanent Inner Crown plus Dais (the configuration fallback). It is a development and balance fallback and is not a player-facing Match mode.
- **Sentry-free Final Fight (fixed).** The Crown Sentries are removed with Layer 3 at about 6:00. **No PvE displacement source may remain active afterward.** A Sentry's push beside a lethal-edged Dais would be a cheap non-player ring-out. A future Warden must not be assumed active in the Dais phase.
- **Crown loot.** From 6:00 to 7:15 the surviving Crown Caches and loose Crown loot stay usable. At the Layer 4 removal the Crown Caches follow the object-cleanup rules, loose Pickups on removed terrain are consumed, and loose loot already on the Dais remains. No loot is teleported. The Dais has no dedicated Cache or objective in v1.
- **Dais edge.** The perimeter stays fully lethal. No railings, invisible walls, safety lips or partial parapets. If the Dais proves unfair, the first lever is the configuration fallback, then deliberate combat or proportion changes, never ad hoc barriers.
- **Ability value on the Dais (accepted).** Tidal Bolt becomes the main ring-out threat, Hook is useful for pulling and setting up but is not the main ring-out ability (it pulls toward the caster), Stone Guard is weaker because a Shielded hit still lands its Knockback, and Arc Burst and Basic Attack stay useful up close. No Collapse-specific Shield or Knockback rule is added.
- **Duration expectation (tunable, evidence only):** a typical Dais fight resolves in about 20–60 s. Repeated fights of 2+ minutes, or frequent Timeouts from the Dais, are strong tuning signals. The Dais can last up to about 2:45 before the failsafe.
- **The Dais is not designed to drop everyone.** The Layer 4 removal should not routinely kill every survivor. The Dais is the validated inward parent and is comfortably reachable during the Warning.
- **Accepted risks:** an early Dais occupant enters the Last Stand with recovered health and ready cooldowns (a positional choice, with no anti-camping rule). Five or six players on the Dais may be chaotic (measured, and the Dais is not enlarged for it). A displacement-dominant final fight is intended up to a point.

#### Tidal publish gate (**measurement-dependent**)

The documentation still contradicts itself on Tidal's push distance (about 5 studs on a tap in the later note, 25 on a tap and 40 charged in the older one).

> **Layer 4 must not be enabled in a publish candidate until tap and charged Tidal displacement have been measured on representative flat ground and on the final Dais greybox.**

- **Measure at least:** tap Tidal from the centre and from common mid-radius positions, charged Tidal from the centre and near the rim, the target with and without movement input, a separate Dash-after-hit recovery test, and Stone Guard (confirm the Shield does not prevent Knockback; if it does, that is a correctness bug to fix before the result is read).
- **Initial operational criterion for "likely lottery"** (measurement-dependent, not a balance law): a single charged Tidal hit on a victim whose `hitRadius` (the **victim's** radial distance from the Dais centre) is about **6 studs or less** can reliably ring them out from otherwise neutral conditions, with no prior edge pressure and no movement mistake. The baseline has the victim stationary or moving ordinarily, with Dash not used. If surviving a centre hit normally requires spending Dash every time, record that as strong evidence the Dais is displacement-dominant even if the strict criterion is not met.
- **The gate is blocking to perform and record, not blocking on a result.** If the Dais is acceptable, Layer 4 may ship enabled. If it is a lottery, this ticket still completes and **Layer 4 ships disabled** (the permanent Inner Crown). Do not redesign Tidal inside this ticket, and do not nerf it silently.
- Telemetry (`chargeBand`, `hitRadius`, `hitDist`) complements the gate and does not replace it.

#### Objects on a removed Section

- **Static objects** inherit the `SectionId` of the marker they were built from: unopened Caches, broken-Cache remnants, Sentries (and any wreck), future Warden slots and any other Section-owned static objective. They are removed with the Section.
- **Loose Pickups** are consumed when, after the geometry is removed, their support ray finds no live floor. The ray uses the same bounded distance as the existing Pickup floor validation. No Pickup needs a Section id.
- **No loot relocation (fixed).** Loose Pickups, whole-Kit drops already lying on the floor and Cache contents are never teleported inward. Loot left on consumed ground is lost, which is intentional scarcity.
- **Order of one removal step:** (1) set the state to Removed and remove the geometry from collision and queries; (2) remove the Section's static objects, cancelling any Sentry Telegraph and pending fire; (3) sweep loose Pickups for support; (4) same-tick interactions see the removed world, and removal wins over a collection or a Cache break completing in that tick; (5) players then fall through the existing Void pipeline.
- **Caches:** every kind is removed with its Section, intact or broken, with no compensation, no automatic inward spawn and no refund. A sealed Crown Cache on removed ground (an invalid configuration) follows the same rule. No special recovery is built.
- **Sentries** stay fully active until their Section is Removed, then vanish at once. There is no early shutdown phase, and no shot may resolve afterward.
- **Projectiles** are combat objects and survive their source Section's removal. They end by their normal rules and see the new world.
- **Rift Wardens** ([Ticket 23](#23-rift-warden)) belong to their slot's Section and despawn with it. A Collapse despawn gives no XP, credit or loot. This step calls `Wardens.removeSection(sectionId)` (idempotent and non-yielding), as it does for the Caches and Sentries, and never destroys Warden instances itself. The Warden loop and its damage gate call `sync(now)`, so a lethal hit accepted before the removal boundary kills normally and one accepted at or after it finds the Warden already gone. A pending Warden impact resolves only if accepted before removal. Wardens are rebuilt from markers after the restore step, like Caches and Sentries.
- **Future persistent world objects** must either carry a Section's id or be removable through the same support check. No generic framework is built for them now.
- **Kit-drop fallback.** A newly generated whole-Kit drop (a `Void` death, or a health death in the removal tick) resolves like this:
  1. Start from the player's `lastGrounded` and apply the existing inward-offset attempt.
  2. Validate against **currently live** floor.
  3. If that ground was consumed, walk the Section's `CollapseParent` chain inward and use the nearest valid floor point on the nearest surviving ancestor. The chain is the only search. There is no navmesh, global nearest-floor search or pathfinding.
  4. Validate the resulting point (and each spread point) against live floor.
  5. If no live floor exists, the Kit is consumed. It is never spawned over void.
  A Kit that was already loose on doomed ground before `T` follows the loose-loot rule and is consumed. A late kill whose loot is lost moments later is an **accepted harsh case**, measured and not softened here.
- The registry caches each Section's bounds, home parent and parent link, so the fallback never needs the removed model in `Workspace`.

#### Match ending, settle window and simultaneous deaths

- **Settle window.** `collapseSettleSeconds` (**measurement-dependent**, 1.5 s as the starting value, **measured and set to 1.75 s in step 9b**, stored as `Tuning.collapse.settleSeconds`) begins at the **applied** removal time, not the scheduled one. `settleUntil = max(settleUntil, removalAppliedAt + collapseSettleSeconds)`. Repeated updates never restart it.
- **No verdict resolves inside the window.** Winner, `NoSurvivors` and Timeout all resolve at `max(existing end-resolution deadline, settleUntil)`, **where `settleUntil` is the window open when the end became pending (amended 2026-11-11, step 9b boundary rule below).** The existing end grace stays and the Collapse adds no result logic of its own. A zero-Alive result cannot change, but it still waits for uniformity, a delay of at most the window. During a pending end the lone survivor cannot act or collect, as today, for at most that long.
- The window is **global**: it is not tied to which deaths were "caused by the Collapse". All deaths count normally during it.
- **After the window,** the existing result rules apply: exactly one Alive wins, none is `NoSurvivors` (a draw), more than one continues. There is no timestamp, last-to-fall, ping, damage, kill-count or combat-credit tiebreak, and no co-winners.
- **Results:**
  - Two survivors lose floor together: `NoSurvivors`, regardless of crossing order.
  - One safe and one falling: the safe player wins only after the window.
  - A combat death and a fall in the same window: if both are dead at resolution, `NoSurvivors`.
- **Attribution is per victim.** A Hook-and-fall pair can end in `NoSurvivors` while the Hooker keeps the kill credit. Winner resolution and elimination attribution stay separate.
- **Harsh trade (accepted):** a player who kills the last opponent and then falls inside the window gets `NoSurvivors`, not a win. It is logged.
- **Boundary rule: a kill before the removal is final (approved 2026-11-11, Ticket 22 step 9b).** A pending end keeps the deadline it was given when it became pending, `max(death + endResolutionSeconds, settleUntil at that moment)`. A removal applied **later, inside the 0.25 s end grace, does not extend it.** A removal applied at or before the death (including the same timestamp, as the Match syncs the Collapse before it asks) has already opened its window, which holds the verdict as above. Both players dying inside the grace is still `NoSurvivors`, at the first death's deadline. Nobody is classified as safe or doomed before a removal. The harsh trade above now applies only to a final kill made **after** a removal was applied.
- **Validation of the value:** it must exceed the measured worst-case server-side Void offset from a player who loses support at the highest relevant point (plus one loop tick and a processing margin), and stay dramatically shorter than the stable gap. It is tuned from the latency measurements, not fixed.
- **Principle:** the window is a correctness fallback. If a future final removal would routinely drop everyone, fix it in the final-Crown design, not by relying on `NoSurvivors`.

#### Runtime architecture

- **`CollapseRules`** (pure) holds config validation, the schedule derivation, next-event derivation, ordering and timing invariants, the shift and settle calculations, the phase-label function, and the HUD presenter (a pure map from attributes and time to a presentation state). It never touches `Workspace`.
- **`Collapse`** (server) holds the Section registry, state mutation, object cleanup, reparenting, telemetry emission and the replicated attributes. It is **driven by `Match`**, in the same way `Caches` and `Sentries` are. Its surface is `start(goAt, matchId, config)`, `update(now)`, `sync(now)` (the gate-facing alias), `settleUntil()`, `stop()` and `reset()`.
- **`Match` stays the authoritative owner** of when Collapse starts, stops and resets and which Match it belongs to. It passes `settleUntil()` into the end check, so `MatchRules` stays pure.
- **Dependencies are passed in:** the map root, holding folder, clock and telemetry sink. That is what lets the specs use a fixture and never touch the real Rift Crown.
- **Match identity.** Runtime state is stamped with the current `matchId`, and a delayed callback from an older Match is ignored and counted. No generic session framework.
- **Ordered cursor and idempotency.** Events are an ordered list with a cursor. The cursor advances only after the event's mutation, cleanup and sweep completed. Operations check state first, and retries never repeat finished side effects, with no transaction framework. Cleanup and telemetry happen once per event.
- **Telemetry never blocks.** It is attempted once inside a protected call. A failure is contained, the gameplay event still completes, the cursor advances, and the failure cannot replay the removal. Do not require a log line proving the logging itself failed.
- **Dormancy.** With the Reference Kit nothing is installed, no Heartbeat step runs and no attributes appear, as with Tickets 17 and 18.
- No multi-session abstractions. No independent service with its own Match lifecycle.

#### Removal mechanism, restore and reset

- **Consumption is reparenting.** At removal each consumed Section or connector model moves from its active home into a **private holding folder** (server-only, under `ServerStorage`). This removes it from collision, queries and rendering at once, and avoids a per-part flag snapshot. Merely setting `CanCollide = false` is not enough.
- **The holding folder holds only consumed models** of the current Match's map and is never a second permanent map hierarchy. Each model's stable home parent is recorded when the registry is built.
- **Restore points (both idempotent).**
  1. **Primary:** at the end of Results, when everyone is back in the Hub: clear the runtime PvE and loot objects, restore the consumed models to their home parents, reset states and attributes, and verify integrity. Restoration must complete before the next Match rebuilds Caches and Sentries from markers. Restoring a large model replicates to every client (`StreamingEnabled` is off), which is why it does not happen at Match start.
  2. **Safety net:** at the start of Match setup, restore and assert again before any marker discovery or character placement. A Match never assumes the previous one cleaned up.
- **Intact-map assertion.** Before a Match can begin: the holding folder is empty, every registered Section and connector is present, every state is `Normal`, no stale removal attribute remains, and the Ticket 21 structural validator still passes. If not, **refuse Match start with a clear diagnostic** and log `CollapseResetFail`. Never start on a partly restored Arena.
- **Caches, Sentries and Pickups keep their lifecycle.** They are cleared at Match cleanup and rebuilt from the authored markers for the next Match. The restore only restores authored geometry and markers.
- **Cached geometry.** Other gameplay code must not hold stale descendant Part references to Sections that may be reparented. The Collapse registry itself may cache identity, metadata, bounds, home parent and parent link. This is checked with a targeted grep at implementation time, not an anti-cache framework.
- **No pristine-map cloning** per Match. Revisit only when concurrent Arenas become real.
- **Studio and Edit-mode safety.** Collapse runtime tests that consume Sections run in **Play**. Any Edit-mode tooling capable of invoking a removal must refuse by default or guarantee restoration in a `finally`-style cleanup. A test run must never save a place with half the Arena in `ServerStorage`. The intact-map assertion is a pre-test and pre-snapshot check.
- **Client effects.** When a Section model disappears, its local veins, beacons and effects clean themselves up, with no orphan at the old location. Any cosmetic debris is spawned client-side from cached bounds and has no authoritative collision or query role.

#### Replicated Match and Section state

Use the existing Workspace-attribute convention. No new remotes are required. Anything routed through a remote must pass `Protocol.validate`.

| Workspace attribute | Meaning |
|---|---|
| `CollapseLayer` | The next relevant event's layer (1–4), or **`0`** for the final-ground sentinel (no further event; presentation only, not a real layer). Absent or cleared before GO and outside Running. |
| `CollapseWarningAt` | Effective Warning start of that event |
| `CollapseRemovesAt` | Effective removal time of that event |
| `CollapseWarning` | Boolean, **true only after the server has actually applied the Warning**, false once that layer is removed |

| Phase | Attributes |
|---|---|
| Before GO, outside Running | absent or cleared |
| Stable, next event pending | `CollapseLayer` = n, `CollapseWarning` false |
| Warning applied | `CollapseWarning` true, `CollapseWarningAt` and `CollapseRemovesAt` effective |
| Layer removed, another follows | next layer's values, `CollapseWarning` false |
| Final ground | `CollapseLayer` = 0, no timestamps |
| Match end | all cleared |

- Each Section or connector model carries only `CollapseState` and `CollapseRemovesAt` for the local and world telegraph. Clients do not receive `CollapseParent`, the registry, cleanup ownership, the cursor or telemetry state. If the escape arrow needs more than the parent's position, expose a small precomputed escape target, not the registry.
- Clients derive presentation from these timestamps every frame, with no local countdown accumulator. A stall-induced shift can make a countdown jump up. That is not a gameplay event.
- Warning style and the world telegraph key from `CollapseWarning` and the per-Section state, never from `CollapseWarningAt ≤ now` alone.

#### World telegraph requirements

Required states and anchors only. Exact materials, colours, particles, meshes and sounds stay with the art pass.

- Every Warning Section **visibly changes**: Section-wide Rift veins or cracks, a rhythmic pulse, and a vertically readable beacon or column that can be identified from another Shard. Effect positions are derived from Section bounds. If that does not reliably give a visible point on an irregular or concave Section, use the smallest robust fallback (for example, a floor sample or a per-Section escape or beacon target). Do not build geometry sampling.
- **Phase lock:** Warning Sections sharing a removal time share the same pulse rhythm, so a pulse never implies a different deadline. Crumble stagger is allowed only around or after removal.
- **Client tiers (derived, not authoritative states):** Early 30–10 s, Urgent 10–5 s, Final 5–0 s (**settled initial value**).
- **Threshold cues:** at 30 s, a brief global announcement and sting with the world Warning; at 10 s, the Urgent presentation; at 5 s, the explicit local final countdown with a per-second cue only for players on Warning ground; at `T`, a positional removal cue. No global per-second ticks.
- **Contrast principle:** Warning ground looks unstable and its inward parent looks stable. No glowing path to safety is drawn.
- **Arrow:** a simple screen-space arrow toward the nearest useful point of the player's Section's `CollapseParent`. Warning connectors get no arrow. If the nearest parent point can point through blocked geometry, use a small authored or derived escape target per Section, not a general navigation system.
- **Accessibility:** colour is never the only channel. Combine text and numbers, pulse rate, pattern, the beacon and audio, and give every important sound a visible equivalent.
- **Mobile and controller:** key HUD information stays in safe top-centre space away from thumb zones, with no hover dependency, no tiny world-only countdown text, and nothing needing a precision cursor.

#### HUD behaviour

One contextual line in the **existing top-centre Match status slot**. There is no permanent second Collapse row, no permanent Match-time row and no phase strip.

| State | Content (format, not final copy) |
|---|---|
| Arrival hold, before GO | `N alive` |
| GO to the first Warning | `N alive · Outer consumed in M:SS` |
| During a Warning | the same sentence with stronger presentation |
| Between removals | the next layer's countdown, such as `Middle consumed in 1:15` |
| Final ground (`CollapseLayer` = 0) | `N alive · LAST GROUND` |
| Last 60 s of the failsafe | `LAST GROUND · match ends in 0:48` (alive count if the layout allows) |
| Results | hidden, as today |

- **Layer names** are Outer, Middle, Approach and Inner Crown, localized. Layer numbers and internal ids are never shown to players.
- **Alive count** stays visible throughout, inline on desktop and on a compact second row if a narrow screen needs it.
- **Gather, Clash, Collapse and Final Fight are not shown to players in v1.**
- **Warning presentation:** the wording stays stable and the styling becomes more urgent: stronger weight, a visible non-colour marker, a pulse synchronized to the authoritative Warning, a faster pulse from 10 s, and a more prominent countdown in the last 5 s. No global per-second audio.
- **Local danger alert:** a separate component **directly beneath** the global line. It never covers it. It shows only while the local player stands on a `Warning` Section, with explicit danger wording, the seconds remaining and the arrow. The duplication is intentional: the global line says what is happening in the Match, the alert says that you personally are in danger. Both derive from the **same** effective timestamp.
- **The alert clears immediately** when the player reaches non-Warning live ground, the Section is removed, the Match ends, or the player stops being an active participant. It never lingers while spectating.
- **Spectators** get the global line and the world telegraphs, with no local alert or arrow, and the existing spectator text must stay compatible. A compact second line is acceptable.
- **Hub viewers** of a Running Match may get the simplified active-Match status, with the next Collapse countdown if it is cheap. They get no alert and no arrow. No separate spectator dashboard is built.
- **Countdown rules (fixed):** whole seconds only, `M:SS` on the global line, plain integer seconds on the alert if that reads better. **Round up:** `displaySeconds = max(1, ceil(removesAt − now))` while that removal is the active next event, and never show `0` or `0:00`. As soon as the server publishes the next event, switch to it, or to `LAST GROUND`. The display rounding is presentation only and implies no extra grace.
- **Failsafe precedence:** the failsafe stays hidden until its last 60 s. If a real Collapse countdown and the failsafe notice ever overlap (an abnormal delay), Collapse wins, and the overlap is logged.
- **The Crown opening** is an event message only.
- **Localization:** entries use named placeholders (for example `{alive}`, `{layer}`, `{time}`) and are never grammatical concatenation. Keys: the alive text, the consumed sentence, `LAST GROUND`, the match-ends notice, the Crown-opened message, the Warning announcement, the local alert and the four layer names.
- **Presenter function:** pure, from the replicated attributes, a synchronized server-time estimate, the alive count, the Match end data and the player or spectator context to one of stable, warning, final, failsafe or hidden, with the layer key, the displayed seconds and the tier. Unit-testable without rendering.

#### Telemetry

Console lines in the existing key=value style. New lines use the `[Riftfall][Collapse]` prefix and carry `match=` and `t=` (seconds since GO). Existing `[Riftfall][Session]` lines stay where they are, and `MatchEnd` and `MatchResult` remain the canonical result. No periodic coordinates, heatmaps, backend, per-frame or per-hit logging, client telemetry framework or new personal data (keep the existing player identifier convention, and log only Section names and radius numbers).

| Line | Content |
|---|---|
| `CollapseStart` (at GO) | player count, schedule, `warnSeconds`, settle seconds, `crownOpenSeconds`, `maxSeconds`, `voidKillY`, whether Layer 4 is enabled |
| `CollapseWarn` | layer, scheduled and applied Warning times, effective removal, delay, cumulative shift, Alive count, Alive players on that layer's ground, `fullKits` and `rank2Players` if cheap |
| `CollapseTick` (10 s and 5 s) | layer, Alive count, Alive players still on Warning ground (and identifiers per the existing convention) |
| `CollapseRemoved` | layer, scheduled, effective and applied removal times, delay, Alive count, Sections removed, static objects removed; loot lost as compact counters (Caches by Home, Forward and Crown, loose Pickups total with Ability and Rank, whole Kits, Sentries); for Layer 4, Alive entering the Dais-only state |
| `CollapseExposed` | one row per player supported by ground being removed, at the removal boundary: identifier, Section, distance to the parent boundary if cheap, recent damage, Stunned, Frozen or Rooted, whether on Warning ground at T−10 or arrived later, and Dash readiness only if Movement already exposes it. A telemetry sample only. It never feeds gameplay and does not mean "dies later". |
| `KitDropRelocated` | emitted only when the live-floor fallback changed the placement: original and target Section, hops if useful, success or failure with a reason |
| `QuietSpan` | with at least 2 Alive and no player-versus-player damage for 45 s, then at most once per 30 s: Alive count, derived phase, Section occupancy counts (no coordinates). PvE and Sentry damage don't count as activity. |
| `CollapseShift` | only when the effective schedule shifts by more than 0.25 s: event, delay, cumulative shift, failsafe margin remaining |
| `CollapseResetFail` | a failed restore, a non-empty holding folder, a failed integrity or structural check, with a concise reason, visible when Match start is refused |
| `CollapseSummary` | one line (or at most a couple of clearly named lines) at Match end, see below |

**Extended existing lines.**
- `Eliminated` gains, where they apply (absent otherwise): `phase` (gather, clash, collapse, final, dais), `sinceRemoval`, `settle`, the displacement source, Ability, charge band (tap or charged), credit age, **`hitRadius`**, **`hitDist`**, and `fallsAfterDash`. `hitRadius` is the **victim's** radial distance from the relevant Arena, Crown or Dais centre **at the moment the displacement hit was authoritatively accepted**, taken from the authoritative combat position and not derived at death.
- `CrownOpened` gains the Alive count, full-Kit count and Rank 2 player count.
- Progression snapshots (Alive, full Kits, Rank 2) are taken at the Crown opening, the Layer 1 Warning and Dais entry. Kit state is never sampled continuously.

**`CollapseSummary` content:** player count, Match duration and end cause; `firstPvpAt` (the first authoritative player-versus-player damage event), `firstElimAt`, the longest quiet span and the quiet-span count; compact `event:alive` checkpoints for the Crown opening, each Warning and removal, and Dais entry; PvP damage and eliminations by phase and cause; Void counts (credited, self, during a settle window); loot loss (Caches, Ability Pickups, Kit drops consumed, relocations, relocation failures); the Dais (players entering, duration, how the Match ended, Dais eliminations, Dais Timeout); runtime health (maximum delay, shift count, resolutions deferred by the settle window, stale callbacks rejected, reset failures). Time between eliminations is derived from the existing `Eliminated` timestamps. A stale callback gets a counter, not a line, with one diagnostic if the count is unexpectedly high.

Telemetry provides evidence. Humans make the "fair or unfair" judgement, and nothing classifies a death automatically.

#### Required configuration values

Everything lives in `Config` (the existing tuning module), not in logic. Names are an implementation decision.

| Key | Start | Label |
|---|---|---|
| Layer rows: `removeAt` for Layers 1–4 | 210, 285, 360, 435 s from GO | tunable |
| `warnSeconds` default (rows may override) | 30 | tunable |
| `warnSeconds` mechanical minimum | 10 (only because the presentation has 10 s and 5 s thresholds. It does not mean 10 s is a good Warning.) | settled initial value |
| First-Warning tuning window | about 150–240 s | tunable (guidance) |
| `firstWarnGuardSeconds` (after the Crown opens) | 90 | tunable |
| `failsafeMarginSeconds` (after the last effective event) | 120 | tunable |
| `Tuning.match.maxSeconds` | 600 | settled initial value |
| `collapseSettleSeconds` | 1.5 starting value, **1.75 measured (step 9b)** | measurement-dependent |
| Settle at most this fraction of the smallest stable gap | 0.25 | tunable |
| Layer 4 row present | yes (omit for the permanent Inner Crown) | settled initial value, with the shipping decision measurement-dependent (Tidal gate) |
| Tier thresholds (Early / Urgent / Final) | 30–10 / 10–5 / 5–0 s | settled initial value |
| `CollapseTick` snapshots | 10 s and 5 s | settled initial value |
| Failsafe notice window | last 60 s | settled initial value |
| `QuietSpan` | 45 s, then every 30 s | tunable |
| `CollapseShift` log threshold | 0.25 s | tunable |
| Dais radius and area (from Ticket 21) | about 14 studs, about 600 studs² | tunable (Ticket 21, greybox-validated), reference only |
| Lottery criterion `hitRadius` | about 6 studs | measurement-dependent |
| Typical Dais fight length | about 20–60 s | tunable (expectation, evidence only) |
| Latency pass margin | 0.25 s | settled initial value |
| Latency settings, blocking / stress | 0, 100, 250 ms / 500 ms | settled initial value |
| Samples per blocking latency setting | at least 5 | settled initial value |
| **Accelerated test schedule** | `crownOpenSeconds` 0, `firstWarnGuardSeconds` 0, `warnSeconds` 15, removals 60 / 100 / 140 / 180 s, `maxSeconds` 300 | test-only |

#### Config validation (`CollapseRules.validate`, runs before a Match starts)

An invalid configuration refuses Match start with a clear reason. It never produces a silent default or an unplayable Match. It rejects:

- a layer outside 1–4, any Layer 5 row, or rows that are not a contiguous prefix from Layer 1 (`[1,3]`, `[1,2,4]` and `[2,3,4]` are invalid, `[1]`, `[1,2,3]` and `[1,2,3,4]` are valid)
- removal offsets that are not strictly increasing
- a non-positive `warnSeconds`, or one below the mechanical minimum
- a Warning that begins before the previous scheduled removal
- a first Warning closer than `firstWarnGuardSeconds` to `crownOpenSeconds`
- `maxSeconds` below the last **effective** removal plus `failsafeMarginSeconds`
- a settle window larger than the allowed fraction of the smallest stable gap
- a scheduled layer with no Section in the active map

#### Structural checks (read-only, safe in Edit mode, blocking)

They also run as the first step of `RunAll`, at Match start (the intact-map assertion) and before a `.rbxl` snapshot is saved.

1. Every scheduled layer has at least one Section.
2. Every `CollapseParent` exists, with a strictly later layer, and every chain reaches the Layer 5 Dais.
3. **Dais isolation (the actual requirement):** after a theoretical Layer 4 removal, no collidable or queryable geometry remains around or inside the intended final arena except Layer 5 Dais geometry and explicitly allowed non-gameplay presentation objects. A presentation part is exempt only if `CanCollide = false`, `CanQuery = false`, and it cannot affect Hook, projectiles or line of sight. This catches terrace or ramp stubs, Rib fragments, invisible collision and ornamental blockers. The Dais must be flat and readable on top, surrounded by clearly lethal faces, and structurally self-contained.
4. No Layer 5 event is scheduled.
5. The holding folder is empty before a Match.
6. The Section and connector registry is complete and every Section is `Normal`.
7. No stale Collapse attributes exist on Sections or `Workspace`.
8. **Every scheduled Section and connector model has exactly one stable recorded home parent suitable for reset.**
9. The Ticket 21 structural validation still passes.
10. **Edit-mode integrity after Play:** after a Play test that consumed Sections, stop Play, inspect Edit mode, rerun the active-map integrity check, and verify nothing from the runtime mutation persisted into the authored map.

#### Pure automated specs (all blocking)

Specs run against a **fixture** map (about 3 Sections with a parent chain, a connector, a marker and static object, and a loose Pickup with support) passed in as the Collapse's dependencies. **Automated specs must never be able to leave the authored Rift Crown partly consumed.**

- **Schedule:** the default is valid with `warnAt` derived. A Layer 4 row omitted is valid. `[1,3]`, `[1,2,4]` and `[2,3,4]` are rejected. Out-of-order or equal times are rejected. A Warning starting before the previous removal is rejected. `warnSeconds` below 10 is rejected, and 30 accepted. The Crown guard and the failsafe margin (against the effective last event) are enforced. A Layer 5 row is rejected. The schedule is the same for rosters of 2 and 6.
- **Runtime:** a Warning applies once. A removal applies once, and only from Warning. Several overdue events apply in order. The cursor advances only after the event completes, and a failed event retries without repeating finished side effects. A stale `matchId` callback is ignored and counted. `CollapseWarning` is false until applied, true during Warning, false after removal, and absent outside Collapse. The `CollapseTick` snapshots fire once each. The Collapse is dormant with the Reference Kit.
- **Schedule-shift chaining (direct regression for the Point 7 amendment):** Layer 1's Warning applies late, its removal shifts, **every later Warning and removal shifts by exactly the same amount**, their 30 s Warning and 45 s gaps remain intact, a Warning and its removal are never applied in the same update, and the failsafe-margin validation uses the final effective event.
- **Settle:** the window starts at the applied removal time, repeated `update` calls never restart it, and `settleUntil` only moves forward.
- **Telemetry failure:** telemetry is attempted once, failure is contained, the gameplay event completes, the cursor advances, and the exception cannot replay the removal later.
- **Removal:** static Section objects are removed. Loose unsupported loot is consumed and supported loot kept. A projectile is not destroyed merely because its source floor disappears. A pending Sentry fire is cancelled and nothing fires after the boundary. Removal beats a same-tick collection or Cache break.
- **Kit-drop fallback:** both branches. With a surviving ancestor the Kit lands on live floor. With none it is consumed. A floor query verifies the location, not just a log line. Spread points are validated against live floor. Displacement records carry `chargeBand`, `hitRadius` and `hitDist`, taken at hit time.
- **Match ending:** one survivor after the window is a `Winner`. Zero is `NoSurvivors`. A combat trade inside the window resolves after it. `resolveAt = max(end grace, settle deadline)`. Raw death-event order never decides the verdict. The existing trade and grace specs still pass unchanged.
- **Reset:** consumed models are restored to their recorded homes, the holding folder ends empty, states and attributes are `Normal` and clear, and marker-driven systems rebuild afterward. A second restore is a no-op. A failed integrity check refuses the next Match start and logs `CollapseResetFail`.
- **HUD and helpers:** the presenter (every state, the round-up rule, never `0` or `0:00`, a stall-induced jump, failsafe precedence), the `QuietSpan` rules (45 s, repeats every 30 s, PvE damage not counted), the phase-label function, and Section occupancy counting.

#### Studio matrix (Play only for anything that consumes Sections)

Timing-critical cases are **armed against an absolute server timestamp** in advance (read `CollapseRemovesAt`, then use a server-armed action or `task.delay` to a fixed absolute time on each client), never by hand through tool latency. The purpose is **server ordering**, not human reflexes. Brief testers per `studio-mcp-quirks.md` (check rendering first, keep calls short, run the most important checks first).

The test configuration is the **test-only** accelerated schedule above, with the guards adjusted through explicit configuration. No production guard is bypassed in code, and the real baseline is never changed to speed up Studio.

**S1: solo Play (blocking, real map).**

| # | Check | Expected |
|---|---|---|
| 1 | Lifecycle | Normal → Warning → Removed per layer, once each, in order. Section and Workspace attributes are correct, and `CollapseWarning` toggles as specified |
| 2 | Removed-geometry invariant, per layer | Character collision, Hook sweep, line of sight and floor raycast no longer see the Section |
| 3 | Stay on Layer 1 until removal | Fall, `Void`, `sinceRemoval` and `settle` logged |
| 4 | Leave at 10 s, 5 s and about 1 s | At 10 s and 5 s, ordinary walking is enough **from a defined position** (a point whose measured walking time to the parent boundary is within the remaining time, with margin). At about 1 s, the outcome is **recorded, not required**: whether it survives depends on the start point. It is evidence for last-second responsiveness, timestamp and UI alignment and boundary behaviour, and is not a map requirement |
| 5 | Fight while the Section is Warning | Basic Attack, Dash, abilities, damage and Pickup interaction all still work until `T` |
| 6 | Pickup at the boundary (absolute-time armed) | A collection before `T` succeeds. One at or after the removal loses to the removal |
| 7 | Cache break at the boundary (armed) | A break before `T` produces loot normally. One in the removal tick or later produces nothing |
| 8 | Sentry Telegraph crossing removal | The Sentry enters its Telegraph before `T`, the Section removes, the Sentry vanishes, and no delayed projectile or damage resolves. Check the log and any stale callbacks |
| 9 | Layer 3 → Final Fight | Sentries absent, no pending Sentry attack, the Inner Crown still valid, Crown Caches still functional, and the global HUD points at Layer 4 if enabled |
| 10 | Layer 4 → Dais | The Dais is the only authoritative ground, terrace loot without live floor is gone, Dais loot remains, no Crown Cache remnants, the HUD shows final ground with no Layer 5 countdown, and no Sentry or PvE displacement source remains |
| 11 | Failsafe | A one-row schedule with a short `maxSeconds` that satisfies the margin shows the notice in the last 60 s, then `MatchEnd cause=Timeout` |
| 12 | Countdown | Derived from the timestamp, no drift over a Warning, the 30 → 10 and 10 → 5 thresholds, the 1 → removal transition, never `0` or `0:00`, a stall-induced shift, and the final-ground transition |
| 13 | Presentation state | Section warning effects appear, tiers transition, removal clears them, no orphan effect remains, and the next Match creates fresh effects |
| 14 | Second Match | See "Reset proof" below |
| 15 | Six-roster sanity | The Collapse and telegraph state function with a six-player roster, the spawn system fits all six, and six rigs or players can stand on the Dais without a geometry problem |

**S2: two clients (blocking).** The developer starts a Server & Clients session.

| # | Check | Expected |
|---|---|---|
| 16 | Fight while both players are on Warning terrain | Nothing is disabled until `T` (as in 5) |
| 17 | Two players fall on different Sections simultaneously | `NoSurvivors`, with the verdict at or after the applied removal plus the settle window, regardless of which plane crossing is observed first |
| 18 | One safe, one falls | The safe player wins only after the settle and end grace |
| 19 | Combat death and a fall in the same window | If both are dead at resolution, `NoSurvivors`. Raw death-event order does not decide it |
| 20 | Hook or Tidal displacement just before removal (armed) | Normal displacement. Credit stands, and the outcome follows the physical state |
| 21 | Whole-Kit drop where the original ground has vanished | Both branches: a surviving ancestor, so the Kit lands on live floor (verified with a floor query, not only `KitDropRelocated`), and no live ancestor, so it is consumed |
| 22 | Dais Last Stand, 2 players | The Dais is the only ground and the fight resolves there |
| 23 | Spectator | The global line and world telegraphs, with no local alert or arrow |

**Reset proof (blocking, the most important check).** Match 1: consume several layers, produce loot, PvE and runtime mutations, and finish the Match. Match 2 must then start, not merely inspect the map: the Arena fully restored, every Section `Normal`, fresh Caches, fresh Sentries, no stale Pickups, no stale Warning attributes, the holding folder empty, the Collapse starting from Layer 1 again, and zero stale callbacks from Match 1 affecting Match 2. Also end one Match mid-Warning and confirm that Warning returned to `Normal`.

**Latency (blocking measurements).** Run at 0, 100 and 250 ms, with 500 ms as stress evidence only, on the observing client, with at least 5 samples per blocking setting from a repeatable armed fall test. Record the server's applied removal time, the client's receipt of the updated attributes, the client's geometry disappearance, the start of the visible and physical fall, the server's Void elimination time, `sinceRemoval`, and the displayed countdown at the authoritative removal. Studio's network simulation is not a model of a real high-ping client (`IncomingReplicationLag` understates what a laggy player sees of targets), so record that limitation.

> **Pass rule:** across the blocking settings, the worst measured **server-side** Void elimination offset for a player who loses support at removal, **plus 0.25 s**, must be at most `collapseSettleSeconds`. If it is not, tune `collapseSettleSeconds`, rerun and record the chosen value. That is a result, not a design failure. Client-side visual delay is measured separately as fairness evidence. If latency makes the last second visually unfair, improve synchronization and presentation first. Do not make client receipt authoritative and do not add a gameplay grace.

**Tidal gate:** as specified above, performed and recorded.

#### Blocking completion criteria

1. All automated specs are green in Play (`RunAll` on the server).
2. The structural and integrity checks pass.
3. The S1 real-map matrix passes.
4. The S2 two-client matrix passes.
5. Reset is proven by actually starting a second Match.
6. The latency settle-margin test passes, after any required tuning of `collapseSettleSeconds`.
7. The Tidal measurement gate is performed and documented.
8. The Layer 4 enable or disable decision follows the Tidal result.
9. Warning and presentation state create and clean up correctly.
10. No stale Match or Collapse callback affects a later Match.
11. Temporary tuning, network settings and test aids are restored.
12. The Edit-mode authored Arena is intact afterward.
13. The Studio and MCP quirks documentation is updated where needed.

#### Balance evidence (recorded, not pass or fail)

These produce tuning notes and do not keep the ticket open:

- whether 3:00 is optimal, and dead time in 2-player Matches
- whether the Collapse creates enough PvP, or just running
- the exact Dais fight duration, and how often charged-Tidal ring-outs happen beyond the publish gate
- loot-loss harshness, and Warning comprehension
- phone, controller and desktop readability, and audio and visual quality (the **state and cleanup** are blocking, the art quality is not)
- a real six-player Dais fight (good, chaotic, cramped or fun)
- real-world settle-window death frequency, and the survival outcome at about 1 s

#### Cleanup checklist (after Studio work)

- Remove the temporary test objects and scripts, and search to confirm they are gone.
- Restore the modified `Tuning` (including `maxSeconds` to the committed baseline of 600).
- Restore `settings().Network` and stop all Play and client sessions.
- Verify the holding folder is empty and the active Rift Crown is intact in Edit.
- Rerun the structural validator and the normal pre-commit checks.

#### Assumptions that no longer apply

- `Tuning.match.maxSeconds = 480` as the baseline. The 480 s figures in Tickets 17–21 and their regression checks are historical.
- A Match's failsafe counted from the start of Running, instead of from GO.
- "The Collapse will not remove sections during play" (Ticket 21's non-goal) and the Ticket 21 "Deferred and unresolved" entries for the Collapse.
- Layer 5 as an undecided, optional layer: it is the Last Stand ground and is never consumed.

#### Known future hooks

- **Unstable stage:** a short non-damaging stage before removal, if evidence shows removal deaths feel cheap.
- **Roster-band first Warning:** an earlier first Warning for small rosters, if 2–3 player Matches show repeatable dead time.
- **Phase labels** in the HUD slot, if new-player testing shows the flow is hard to follow.
- **Overtime for the Dais** (candidates: authored Dais rings, a combat modifier), only on evidence that Dais Timeouts are not rare.
- **Wardens:** their own rule for the Dais slot and for despawn on consumption.
- **Cosmetic debris** from cached Section bounds, and a final art, audio and palette pass.
- **Concurrent Arenas:** revisit cloning and per-session Collapse state.

#### Deferred and unresolved

- The final visual and audio treatment of the telegraph, the beacon, the sting and the tick. (Staged in [Ticket 26](#26-rift-collapse-visuals-and-atmosphere), added 2026-11-10. This ticket still builds the minimal functional telegraph below.)
- Warden stats, AI, XP, drops and credit, and their behaviour in the Last Stand.
- Whether a dead mid-game or Dais stall justifies any new mechanic.
- Whether the Layer 4 timing, the 30 s Warning and the 45 s gap suit real play (tuned from telemetry).
- ~~The final value of `collapseSettleSeconds` (set by the latency measurement).~~ Resolved in step 9b: 1.75.
- ~~Whether the Dais is acceptable for Tidal, which sets whether Layer 4 ships enabled.~~ Resolved in step 9b by the developer (2026-11-11): Layer 4 ships enabled.

#### Integration contract for Ticket 25 (added 2026-11-10)

[Ticket 25](#25-match-milestones) (Milestones, **Stage 2**) reads from this build. These are read-only requirements, with no change to the schedule or the Section rules:

- A **Section lookup from a position**, and each Section's **Warning flag**, available to server code (the `CollapseWarning` state already exists for clients).
- The **applied** removal time and the end of the settle window for each removal, so Pressured Exit uses the same stalled-or-shifted time the Collapse used.
- (Stage 3 only) the Kit-drop live-floor fallback must handle **up to three** dropped items.

#### Integration contract for Ticket 26 (added 2026-11-10)

[Ticket 26](#26-rift-collapse-visuals-and-atmosphere) (Collapse presentation, **specified, not built, staged**) builds on this ticket's replicated state and **changes no gameplay contract**: not the schedule, atomic and phase-locked removal, collision, the removed-geometry invariant, the settle window, the server-authoritative timing or the optional Layer 4. These are the points to keep in mind when building this ticket:

- **Replace, don't duplicate.** This ticket still builds its **minimal functional** world telegraph and placeholder cues, and its criterion 9 and matrix rows still apply. When Ticket 26's Stage 1 is built, it **replaces or extends that client presentation in place**, in the same module and on the same attributes, rather than adding a second controller or duplicate warning effects and audio. Keep the client telegraph in as few modules as practical so that swap stays small.
- **State Ticket 26 reads, so it must stay as specified here:** `CollapseLayer`, `CollapseWarning`, `CollapseWarningAt` and `CollapseRemovesAt` on the Workspace, and `CollapseState` and `CollapseRemovesAt` on each Section or connector model. A Section leaving its home parent at removal is a signal Ticket 26 relies on, and the holding-folder ancestry behaviour is checked at build.
- **No new `CollapseState` transition is added for Ticket 26** unless its testing proves the existing signals insufficient.
- **Authored Section `CollapseLayer`** must be readable by clients.
- **Cue semantics stay:** the 30 s announcement and sting, the Urgent presentation at 10 s, the per-second cue only for players on Warning ground in the last 5 s, a positional removal cue, and no global per-second ticks.
- **Cosmetic debris and the art pass** listed under Future work belong to Ticket 26, client-side and non-interactive, with no authoritative collision or query role.

#### Dependencies, risks and ADRs

- Depends on Ticket 21 (see above) and on Tickets 15–20. After the swap to the real Rift Crown, the Ticket 20 regression must still pass with `maxSeconds` at the new baseline.
- **Risks:**
  - Consuming authored models is a persistent-state hazard in Edit mode, covered by the Play-only rule and the pre-snapshot assertion.
  - Restore replicates a large model to all clients, covered by restoring at Results end.
  - Any system that caches Section parts breaks silently, covered by the targeted grep.
  - The Dais may be a Tidal lottery, covered by the gate and the configuration fallback.
- **No ADR required.** The Collapse's design is tunable gameplay and a Rules plus system implementation, recorded here and in the design archive. Reparenting, the atomic update and the fixed schedule are reversible, and ADR-0002 (movement), ADR-0005 (Studio workflow) and ADR-0007 (one life) are unaffected. Revisit only if concurrent Arenas or a damage-based Collapse ever become real.
- **Glossary:** `CONTEXT.md` gains **Warning**, **Consumed** and **Last Stand**, and the Match, Rift Collapse and Section entries are corrected (the "about 12 minutes" figure is removed, since it contradicted `maxSeconds`).

#### Implementation notes (not part of the gameplay specification)

- Follow `CLAUDE.md`'s division: the main session builds the runtime and anything touching authority (`CollapseRules`, `Collapse`, the Match integration, the Kit-drop fallback, the settle gate). `studio-implementer` writes the mechanical specs from the written list, and `studio-tester` runs the Play matrices and returns compact tables. The developer launches multi-client sessions.
- Read `docs/testing/studio-mcp-quirks.md` first. Run `RunAll` on the server in Play, because Edit-mode `require` can return stale modules. Read Session lines through `LogService:GetLogHistory()` on the server.
- **Build order:** (1) `CollapseRules` with its specs. (2) The fixture and the `Collapse` runtime with injected dependencies, with removal, cleanup and reset specs. (3) The Match integration: the gates' `sync`, the settle gate in the end check, `maxSeconds` and the `goAt` re-base, the start, stop and reset call sites, and the intact-map assertion. (4) The Section and Workspace attributes. (5) The Kit-drop fallback, the displacement-record fields and the extended `Eliminated`. (6) The HUD presenter, the `MatchHud` line, the local alert and the telegraph effects with their cleanup. (7) Telemetry. (8) The structural checks. (9) Swap onto the real map and run S1, S2, latency and the Tidal gate. (10) Cleanup, documentation and the commit. Each step is testable before the next.

#### Review notes (2026-11-09)

An adversarial pass against Ticket 21, the settled design points, the existing Match-end and Void semantics, the HUD conventions and the Rules plus system architecture found **no blocking contradiction**. Fixes applied to the documents:

- **The Match-end wording:** the settle window defers **all** verdicts (Winner, `NoSurvivors` and Timeout) uniformly, which slightly extends the old "zero Alive ends at once" rule by at most the window. The ticket now states it.
- **"Authoritative physical state" and ADR-0002:** movement is client-driven and server-validated, so the ticket says the plane crossing is the authoritative record of a fall and that the server never pre-classifies.
- **The `maxSeconds` baseline** moved from 480 to 600 and is re-based on `goAt`. The 480 s mentions in Ticket 21 now point here, and the historical regression text is declared historical.
- **Stall handling** was made general: any late event shifts all later events by its delay, which covers a late removal as well as a late Warning.
- **The glossary** had an "about 12 minutes" Match that contradicted `maxSeconds`, and the Section entry described a future Collapse. Both are corrected.
- **Older "no Rift Collapse" statements** in this file (the Ticket 17 scope and the Match end and timeout note) and `match-structure.md`'s 9–12 minute Collapse phase now carry dated notes pointing here. The long-term phase times stay as a target for a larger mode.

Accepted, with evidence to follow, and for the developer to veto at review:

- **Crown Caches sit about 5 studs inside the post-Layer-3 lethal lip.** Ticket 21's lip-margin rule covers the intact map only. Final Fight Void deaths near the Crown Caches are visible through `Eliminated` with `phase=final`, `hitRadius` and `hitDist`.
- The harsh trade, a late kill's lost loot and early Dais occupancy, all measured.

Correctly deferred: Warden behaviour, final art and audio, overtime, roster-band timing, phase labels, an Unstable stage and concurrent Arenas.

Review result: **no ADR required.**

#### Build record (2026-11-11, started)

**Step 1, `CollapseRules` (done, 545 specs: 498 + 47).** Pure module in `Shared/Rules`, config in the new `Tuning.collapse`, specs in `CollapseRulesSpec`. Nothing in the game requires it yet, and `Tuning.match.maxSeconds` is still 480 (it moves to 600 with the `goAt` re-base in step 3, because the Layer 4 row fails the failsafe margin against 480).

Decisions taken while building, for review:

| Decision | Why |
|---|---|
| `Tuning.collapse.rows` holds `{ layer, removeAt, warnSeconds? }` only. `CollapseRules.rows` derives `warnAt` | One source of truth, as specified |
| `peek(state, now)` offers the next event if due, and `complete(state, event, now, config)` advances the cursor, adds the delay to the shift and extends `settleUntil`. A repeated or stale `complete` returns nil | Lets the runtime mutate, clean up and sweep between the two, and retry a failed event without repeating the finished ones |
| The state holds a cumulative `shift`, and an event's effective time is `goAt + offset + shift` | A late event delays every successor by exactly its delay, including its own removal |
| The next removal's time is published as soon as the next event is known, with `CollapseWarning` false until the Warning is applied | The HUD's "Outer consumed in M:SS" needs it before the Warning. The ticket's attribute table lists the times only for the applied Warning, so this is a small extension for the developer to veto |
| `CollapseRules.present` returns `hidden`, `stable`, `warning`, `final` or `failsafe`, plus `layerKey`, `seconds`, `tier`, `alert`, `lastGround`, `failsafeSeconds` and `overlap`. `final` is the LAST GROUND state, and the tier (`early`, `urgent`, `final`) is separate | Matches the ticket's presenter description. Localization stays in `MatchHud` |
| `phase` returns `final` after the Layer 3 removal, or after the last row when there are fewer than three, and `dais` only with a Layer 4 row | Keeps the permanent Inner Crown fallback labelled sensibly |
| An empty schedule is rejected, and so is a first Warning before GO | An unplayable schedule should never start silently |
| A Warning may begin at the previous removal (zero stable gap), only before it is rejected | The ticket says "begins before". The settle-fraction rule then needs `settleSeconds` 0, and a spec covers it |
| Also pure, for later steps: the `QuietSpan` state, Section occupancy, `resolveAt` for the settle gate, and `liveAncestor` for the Kit-drop `CollapseParent` walk | Each is in the ticket's list of things the pure module and specs must cover |

Two of my own spec expectations were wrong at first (the gap after a shift is still 30 s, and 11 s left is still `early`). Both were fixed in the spec, and the rules were not changed.

**Step 2, the `Collapse` runtime on a fixture (done, 574 specs: 545 + 29, 0 failed).** `ServerScriptService.Server.Collapse` is a factory, `Collapse.new(deps)`, so a spec runs it on a fixture and never touches the real Rift Crown. Only `CollapseSpec` requires it: `Match` and every other system are unchanged, `Tuning.match.maxSeconds` is still 480, and with the Reference Kit nothing is installed. Wiring it into `Match` is step 3.

What it holds:

- **Registry.** Built from the map folder at `new`: every Section (all five layers, so the parent chain is complete) and every connector with geometry, each with its one recorded home parent, layer, `CollapseParent` and bounds. A Dash shortcut is a marker Part with no geometry and is skipped. Layer 5 is registered but never scheduled.
- **Removal.** One non-yielding step per layer, in the ticket's order: models go `Warning → Removed` and are reparented into the holding folder, then `objects.removeSection(id)` runs once per removed Section, then `objects.sweepPickups(layer)`. The cursor advances only after all of it, and a failed event is retried by the next `update` without moving a model twice.
- **State.** `CollapseState` and `CollapseRemovesAt` appear on a model only while it is `Warning` or `Removed`, and are absent when `Normal`. The Workspace attributes come from `CollapseRules.publicState` after each event, on an injected holder.
- **Lifecycle.** `start` (refused outside Play, on a map that isn't intact, on a running Collapse or on an invalid configuration), `update(now, matchId)` and its alias `sync`, `stop` (a Warning returns to Normal, consumed models stay held), `restore`, `prepare` and `reset` (restore, then assert, and a failure logs `CollapseResetFail`), `assertIntact`, `settleUntil`, `stats`, `liveAncestor` and `sectionsIn`.
- **Telemetry** goes through one injected sink inside a protected call, after the cursor has moved: `CollapseStart`, `CollapseWarn`, `CollapseRemoved`, `CollapseShift`, `CollapseResetFail`. Step 7 adds the Alive and loot fields and the remaining lines.

Verified against the developer's step 2 clarifications:

- **Early publication is inert.** The next removal time is published with `CollapseWarning` false and no model is marked until the Warning is applied (spec: start publishes layer 1, nothing is `Warning` at GO + 44.9). Clients must key every ground-warning effect, sound and Riftlight visual from `CollapseWarning` and the per-model `CollapseState`, never from the timestamp (the Ticket 26 contract already says so).
- **Zero stable gap.** A removal and the next Warning at one timestamp apply in order, each to its own layer, in one update (`CollapseRemoved:1` then `CollapseWarn:2`), none skipped or doubled. With a late removal the next Warning shifts by the same delay and is then due at once, so it applies in the same update with no delay of its own and stamps the shifted removal time on layer 2's models. A spec covers each.
- **Simultaneous and stalled.** A layer's Sections and connectors are all `Removed` before the first object hook runs, and layer 2 is untouched. A late Warning stamps models and Workspace with the shifted removal time, and a 155 s stall still gives a full 30 s Warning.

Specs (29, on a fixture of three Sections with a parent chain, two connectors, a Dash marker, hooks and a state holder, all destroyed after each test): building is inert; the registry; start publishing; Warning once; removal by layer; atomic removal; the removed-geometry invariant against a real floor raycast, including inside the sweep; hooks once per removed Section; logged counts; the two zero-gap cases; a stalled Warning; a long stall; settle start and no restart; stale callbacks before, during and after; telemetry failure contained; a failed event retried without repeating; the final ground and Layer 5; the parent chain; an interrupted Match; restore and its idempotence; a second Match after reset; start refused when not intact; stray holding models, stale attributes and the structural problems in the integrity check; a destroyed model reported; an invalid configuration and a second start refused; nothing runs outside Play; the clock default; and a last spec that the fixture is gone and the real map untouched.

Decisions taken while building, for review:

| Decision | Why |
|---|---|
| The holding folder defaults to a Folder in `ServerStorage` that the instance creates, and the specs pass their own | Server-only, and nothing outside the instance owns it |
| `restore` ends a running Collapse first | A restore with a live schedule would fight the cursor |
| `start` does not restore: it asserts and refuses. `prepare` (restore, then assert) is the Match-setup safety net | Matches the ticket's two restore points, and a start never hides a half-restored map |
| A model that is `Normal` at its removal event is skipped and counted (`invalidTransitions`), never removed | Only `Warning → Removed` may happen at runtime. It cannot occur while the cursor is ordered, so a non-zero count is a bug |
| The failure of an event is logged once per event, with `warn`, and retried on every later `update` | A stuck event must be visible without flooding the log |
| A retried removal reports the whole layer's counts, taken from state | The first attempt had already moved the models |

Two of my own spec expectations were wrong at first and fixed in the spec: the late zero-gap case (above), and a fixture raycast that has to respect `CanCollide` because the Dash marker overlaps a slab. No behaviour in the runtime changed because of either. I did change the Warn log to count Sections and connectors separately and the removal log to count from state, after reviewing the specs.

State after step 2: Edit mode shows the map intact (15 Sections, 12 connectors, 29 markers, 6 pads, no stale attribute), no temporary object, `startingKit = "Reference"`, `maxSeconds = 480`.

**Step 3, the `Match` integration (done 2026-11-11, 601 specs: 574 + 27, 0 failed).** The Collapse now runs in a real Match. With the Reference Kit nothing changes: no Collapse, holding folder, attribute or loop exists (checked in Play after `RunAll`).

Settled with the developer before building:

| Point | Decision |
|---|---|
| The pre-start check fails (the map isn't intact after the restore, or `Tuning.collapse` is invalid for the map) | **Disable Matches for good**, one loud error with the reason, as for an uneven loot bag |
| `Collapse:start` fails at GO although the pre-start check passed (should be impossible) | **Play on without a Collapse**, one loud error, and the 600 s failsafe ends the Match |

What changed:

- **`Tuning.match.maxSeconds` is 600.** The failsafe was already counted from GO (`MatchRules.arrive`, Ticket 21), so the re-base needed no code, only a spec that the deadline is `goAt + maxSeconds`.
- **`Match`** builds the Collapse from `deps.collapseFactory` (`Collapse.new`, passed by `Main`) only with an Empty Kit and a Rift Crown in the place. It validates the schedule once at startup through the new `Collapse:validate` (a schedule that fails disables Matches). It then:
  - runs `prepare` (the safety-net restore and the intact assertion) just before the loot deal, so a refusal changes nothing in the world
  - calls `start` at GO, right after `caches.setGo` and `sentries.setGo`
  - calls `stop` first thing in `endMatch`, and `reset` in `endResults` after the Pickups, Caches and Sentries are cleared (a failed reset is reported loudly, and the next start's `prepare` refuses on it)
- **The boundary.** `Match.sync` (`collapse:sync(clock(), matchId)`, only between start and stop) is called by `canAct`, `canCollect`, `canBeHit` (the target filter), the Cache attacker gate, `tick`, every Sentry loop pass and each Sentry's own step, and by a Heartbeat connection. With nothing due it only looks.
- **The settle gate.** `MatchRules.checkEnd(state, now, config, settleUntil)` and `step`'s `inputs.settleUntil` make every verdict wait for `CollapseRules.resolveAt(its own deadline, settleUntil)`:
  - Winner and `NoSurvivors` wait. A window that opens after the end grace began pushes the resolution out.
  - A Timeout waits too, and is not a pending end, so nobody is stopped from acting.
  - `Match` schedules the resolution at the window's end, so it lands without the loop.
- **Hooks.** `Caches.removeSection(id)` removes every Cache on a Section, intact, sealed or broken, with no drops. `Sentries.removeSection(id)` removes the Sentry or its wreck, cancels a Telegraph and its pending shot (`stepSentry` finds the record gone), and `Sentries.setSync` installs the sync. `Pickups.sweepUnsupported(layer)` consumes every loose Pickup with no live floor under it, silently, and counts total, whole-Kit drops and each Ability and Rank. Each returns flat counts, and `Match` prefixes and sums them (`objects.caches_Home`, `objects.sentries_total`, `pickups.kits`).
- **Markers name their Section.** `Caches` and `Sentries` read `SectionId` from their markers, which already carried it from Ticket 21.
- **A minimal telemetry sink** prints each Collapse event as one `[Riftfall][Collapse]` line of sorted key=value pairs. Step 7 adds the Alive and loot fields.

Decisions taken while building, for review:

| Decision | Why |
|---|---|
| A Cache hit re-checks `record.removed` after the attacker gate | The gate syncs the Collapse, which can remove that very Cache mid-hit, and a break completing in the removal tick must give no loot |
| `Pickups.sweepUnsupported` has its own support ray with `RespectCanCollide = true`, injected as `isSupported`, and leaves `findFloor` as it was | A non-colliding marker Part (a Dash shortcut) must not count as floor, and Pickup drops stay exactly as before |
| With nobody Alive inside the window, `checkEnd` sets `pendingEndAt` to the window's end | It lets `Match` schedule the resolution. Nobody is left to be stopped from acting |
| `Collapse:validate(config, ctx)` was added and `start` uses it | `Match` needs the schedule checked on the real map at startup, and the check must be the one `start` runs |
| `Match` takes a factory, not an instance | With the Reference Kit no Collapse, holding folder or registry may exist |

**Specs (27 new).** `MatchRulesSpec` (5): each verdict waits for `settleUntil`, the window opening after the grace, zero Alive, Timeout, and `step`'s input. `MatchSpec` (12, on a fake Collapse): built only with an Empty Kit, an invalid schedule disables Matches, the objects hooks, the start order and the 600 s `maxSeconds`, a map that isn't intact refuses the start with nothing set up, a failed start at GO plays on loudly with no sync, every gate and the loops sync for this Match, nothing syncs before the start or after the end, the settle window with and without the loop, a removal after the grace began, stop before the result and reset after Results, and a failed restore. `CachesSpec` (4), `SentriesSpec` (4) and `PickupsSpec` (2): the three hooks, their idempotence, the broken-Cache and wreck cases, the Telegraph, the sync, and the sweep.

**Live smoke on the real Rift Crown (accelerated schedule, then reverted).** One player, an Empty Kit, removals at 45, 80, 115 and 150 s, `crownOpenSeconds` 0 and `maxSeconds` 300. The first Match: `CollapseStart`, `CollapseWarn` at t = 30.00 (six Sections and a connector), `CollapseRemoved` at t = 45.00 with 12 Home Caches removed and 7 models held, a `Void` elimination at 46.1 s (the player stood on Layer 1), and `MatchEnd` `NoSurvivors` at 46.5 s, exactly the 1.5 s settle window after the removal. At the end of Results the map was back to 15 Sections, 0 Caches and an empty holding folder, and Match 2 then passed the pre-start check and started. In Match 2, two loose Pickups on Layer 1 and one on the Dais went through the real `sweepUnsupported` after the removal: the two on the removed ground were consumed and the Dais one stayed. **Not yet exercised live:** Layers 2 to 4, a Sentry removal and a survivor through a removal. They belong to step 9.

**State after step 3.** Edit mode shows the map intact (15 Sections, 12 connectors, 29 markers, 6 pads, no stale attribute), no temporary object or `TEMPORARY` text in `Tuning`, `startingKit = "Reference"`, `maxSeconds = 600`. The grep for code that caches Section descendants found none: `Characters` caches pad Parts (the `Pads` folder, never consumed) and `MapLayout` caches plain data. I also deleted 8 stale, empty `PickupsSpecFolder`s from `ServerStorage` that were left from earlier Edit-mode spec runs. Until step 5, a Kit dropped after a `Void` death validates against live floor only (the removed ground no longer counts), with no `CollapseParent` fallback.

**Steps 4 and 5, the attributes and the Kit-drop fallback (done 2026-11-11, 622 specs: 601 + 21, 0 failed).** Built together, because step 4 turned out to be a check with one fix.

**Step 4, the attributes.** Checked against the ticket's table, row by row. Everything matched except one thing: `Collapse:start` published `CollapseLayer` and the times as soon as it was called, which is during the arrival hold, before GO, where the table says absent. It now publishes at GO: `start` takes the caller's clock in `ctx.now` and publishes only if GO has come, and the first `update` at or after GO publishes otherwise. `Match` passes `now = clock()`. The final-ground row (`CollapseLayer = 0`, no timestamps), the Match-end clearing and the per-model `CollapseState` and `CollapseRemovesAt` needed no change. A spec covers the deferral, a stop and a second start before GO. Whether the Section models should carry a precomputed escape target for the arrow is left to step 6, which is where it is used.

**Step 5, the Kit-drop fallback and the extended `Eliminated`.**

- **Fallback.** A whole-Kit drop is first worked out as in Ticket 21. If that point has live floor, or was never over a consumed Section, nothing changes. Otherwise `Collapse:relocate` walks the Section's `CollapseParent` chain inward and takes the nearest live floor on the nearest surviving ancestor:
  - the point on the ancestor's box nearest the drop, pulled in `kitDropInwardStuds` (6) from the edge, then stepped towards the box's centre in 16 steps until a ray finds floor on that ancestor's own model
  - an ancestor with no floor passes to its own parent, and a chain that runs out **consumes the Kit**, which is never dropped over the void
  - each spread point is checked against live floor, as Ticket 21 already did
  - `KitDropRelocated` is logged only when the placement changed, with the original and target Sections, the hops, `result=relocated|consumed` and the reason. A consumed Kit is still announced to clients from where its owner was.
- **Displacement records.** `Casts` passes the Cast's charge band to its delivery, `Projectiles` and `Sectors` carry it to `Combat.applyHit`, and `Combat` gives the displacement listener `{ chargeBand, position, distance }`: the band of the Cast, the victim's own root as the hit was accepted, and the displacement the server gave (`knockback.Magnitude * its duration`). `Match` turns the position into `hitRadius` (the victim's flat distance from the map's centre) and stores all three in the credit, so they are what the server knew at hit time and are never derived at death.
- **`Eliminated`.** After a credited fall the line gains `ability`, `chargeBand`, `hitRadius` and `hitDist`. During a Collapse every elimination gains `phase`, `sinceRemoval` (absent before the first removal) and `settle=yes|no`. Each is absent where it doesn't apply, so a self fall, a Sentry's push, an expired credit and every non-Void line read as before.

Decisions taken while building, for review:

| Decision | Why |
|---|---|
| `hitDist` is the displacement the server gave the victim, in studs, and not the victim's distance to the edge | The ticket names the field without defining it. With `hitRadius` it says where the push started and how far it went, which is enough to judge a ring-out without position sampling. Say if you meant the distance to the lip |
| Bounds in the registry are **world-aligned** boxes, built from the corners of each model's own bounding box | `Model:GetBoundingBox` follows the pivot, which on the real map is rotated (the Dais reads 29 tall along its X axis), so "local Y is up" was false. Found by the live run below, and a spec now covers a rotated Section |
| Which consumed Section a point was over is the one whose box contains it with the **nearest centre** | Neighbouring Sections' boxes overlap, and a shard's box is coarse (about 96 by 118 for Shard 1) |
| The floor under a candidate is the **lowest upward-facing surface within 12 studs** of the highest in its column | A Rib's top is a surface, and a Kit left 5 studs up on it can't be collected from the terrace beside it. Found by the live run, and a spec covers it |
| `Pickups`' own `findFloor`, `VoidRules.dropOrigin` and the eliminated-position fallback are unchanged | The fallback is a layer on top of the Ticket 21 answer, so where floor still exists the behaviour is exactly Ticket 21's |

**Specs (21 new).** `CollapseRulesSpec` (1): `sinceRemoval` and `inSettle`. `VoidRulesSpec` (1): the credit's three fields. `CombatSpec` (1): the listener's info. `SessionSpec` (2): the extra `Eliminated` fields and their absence. `SectorsSpec`, `ProjectilesSpec`, `CastsSpec` (1 each): the band reaches the hit, a tap is `Tap` and an Instant Cast has none. `CollapseSpec` (7): the deferred publish, `timing`, `removedSectionAt`, relocation to a surviving ancestor over one and two hops, consumption with no live floor, a rotated pivot, and a wall's top. `MatchSpec` (6): a credited fall's fields, none on a self fall or an expired credit, timing only while a Collapse runs, relocated drops, a consumed Kit, and unchanged behaviour where floor exists or the ground was never consumed.

**Live check on the real Rift Crown (accelerated schedule on a second `Collapse` instance in Play, then restored).** Layers removed at 12, 30, 48 and 66 s, with a drop point sampled on Shard 1's Outer Section, Shard 3's Inner Section and the Crown's Outer ring after each:

| After | From | Landed on | Hops |
|---|---|---|---|
| Layer 1 | Shard 1 Outer | `Shard1Inner.Slab` | 1 |
| Layer 2 | Shard 1 Outer and Shard 3 Inner | `CrownOuter.BandFloor` | 2 and 1 |
| Layer 3 | Shard 1 Outer, Shard 3 Inner and the Crown Outer ring | `CrownInner.Terrace` | 3, 2 and 1 |
| Layer 4 | Shard 1 Outer and the Crown Outer ring | `CrownDais.Dais` | 4 and 2 |

The first two runs found the rotated-pivot bug (every relocation answered `NoLiveAncestor`) and the Rib top, and both were fixed before the table above. After each run `restore` put every model back and the integrity check passed. **Not yet exercised live:** a real Match through a `Void` death with a Kit that is relocated, and the `Eliminated` lines from a real fight. Both belong to step 9.

**State after steps 4 and 5.** Edit mode shows the map intact (15 Sections, 12 connectors, 29 markers, 6 pads, no stale attribute), `ServerStorage` holds only `Tests`, `Tools` and `MatchMapGreybox`, `startingKit = "Reference"`, `maxSeconds = 600`, and with the Reference Kit no Collapse or holding folder exists.

**Step 6 settled with the developer (2026-11-11), before building.**

| Point | Decision |
|---|---|
| Escape arrow | Point at the nearest point of the `CollapseParent`'s own geometry first. Check the real map in Play for Sections where that points through blocked geometry, and add a per-Section escape target only for those |
| `hitDist` | Kept as built: the displacement in studs the server gave the victim, not the distance to the lip |
| Audio | None in step 6. Every cue gets its visual equivalent, and sound waits for Ticket 26 (no upload, no built-in sound risk) |
| Viewers | The same global line for everyone. Hub viewers get the countdown in `Match in progress` text, spectators get the global line with no alert or arrow, and the spectator bar keeps its own alive count |

**Step 6, the Collapse's client presentation (done 2026-11-11, 636 specs: 622 + 14, 0 failed).** The HUD line, the local danger alert with its arrow, the Warning announcement and the world telegraph, all client-side and all reading the attributes step 4 publishes. No server code, no new remote, no sound.

What was built:

- **`CollapseRules` (pure) gained the presentation rules:** the player-facing text table with `{named}` placeholders (`TEXT`, `format`, `layerName`), `statusLine` (the global line for any presenter result and context), `alertLine`, `announceLine`, `pulse` (0 to 1 from the seconds left and the tier) and `arrowAngle`. `Tuning.collapse` gained `pulseHz` and a `display` table. `Protocol` gained the five `COLLAPSE_*` attribute names, and the server still writes literals.
- **`MatchHud`** takes the status line from `CollapseRules.present` whenever the Workspace carries `CollapseLayer`, and is unchanged otherwise (the Reference Kit, the arrival hold, Waiting, Countdown, Results). A Warning keeps the sentence and adds a `!! ` marker, an amber pulse and a larger size by tier. The local alert sits directly under the line, shows only while the presenter says `alert`, and carries the seconds and a rotating arrow. The Warning announcement shows for 4 s. The line, the alert and the pulse update every frame from the timestamps and are dormant when no Collapse state exists.
- **`CollapseTelegraph`** (new, client) gives every Section or connector whose own `CollapseState` is `Warning`, while `CollapseWarning` is true, a Riftlight `Highlight` and a vertical neon beacon, both pulsing on the removal time so Sections sharing one removal share one rhythm. Every effect is a client-local instance in `Workspace.CollapseTelegraph`, non-collidable and non-queryable, destroyed when the model leaves Warning or the world or the Match changes. It also answers which Section the local character stands on (a downward ray against the map every frame while a Warning is applied) and the arrow's direction. It is the module Ticket 26 replaces in place.

Decisions taken while building, for review:

| Decision | Why |
|---|---|
| The status line is the presenter's for all three contexts, and a spectator (`Eliminated`) now sees it, where the old line was hidden | Settled above. The spectator bar keeps its own alive count |
| A Warning connector gets the alert and no arrow | The ticket says connectors get no arrow, and a connector is a Section for state |
| The alert shows only for a character within 16 studs above a Warning Section's floor | One ray down. A long Dash over a Warning Section may blink it for a moment |
| The beacon stands on a floor sample: downward rays against the whole map through rings of the Section's largest parts, accepting the first upward-facing hit that belongs to the Section | The Crown is stacked discs (Band, Terrace, Dais). A ray through the centre of CrownOuter or CrownInner hit buried geometry, which would have put the beacon inside the Dais and hinted that the Dais was in danger. This is the ticket's floor-sample fallback, not geometry sampling |
| The arrow's target is the nearest point of the parent's own collidable parts by horizontal distance, with a cylinder taken as a disc | A box round a cylinder is off by up to 10 studs at a corner. No per-Section escape target was needed on the real map (settled: direct first) |
| The Warning announcement fires only for a layer this client watched in its stable state first | A joiner mid-Warning sees the line and the telegraph but no banner. It also means a zero stable gap would not announce, which the default schedule never has |
| The HUD holds its last presentation for 80 ms after any Collapse attribute changes | The four attributes arrive a few frames apart. A sampler at the first removal saw the next layer with the last layer's countdown and a spurious announcement, and this removed both |

**Specs (14 new, all in `CollapseRulesSpec`):** `format` and an unknown placeholder, the line before the first event, each layer's name and M:SS countdown, the Warning marker, never `0:00`, the final ground, the failsafe notice and the Hub and spectator wording, no layer number or Shard id in any line, the alert and announcement texts, the line and the alert showing the same seconds from the same timestamp, the pulse depending only on the time left, its range, rate and trough, its continuity across the 10 s and 5 s tiers, and the arrow angle in both camera orientations and for no direction. The client modules have no specs, because specs run on the server, and are covered by the live checks below.

**Live checks (real Rift Crown, Play, the accelerated schedule with a 30 s Warning, then reverted).** A frame-by-frame client sampler read the HUD and the effects folder:

| Check | Result |
|---|---|
| Before GO and with no Collapse | Old status line, no alert, no effects |
| GO | `1 alive · Outer consumed in 1:00` |
| Warning applied | The line gains `!!`, the announcement `OUTER IS BEING CONSUMED` shows for 4 s, the alert appears at once with `30` and an arrow, and 14 effects appear (7 models, a highlight and a beacon each) |
| Countdown | Line and alert show the same whole seconds from the same timestamps |
| Removal | The alert clears on the removal, the effects go to 0 in the same frame, and the line moves to `Middle consumed in 0:40` with no spurious announcement (after the 80 ms hold) |
| Spectator after the fall | `0 alive · Middle consumed in 0:39`, no alert |
| Results and the next Match | Everything hidden, `fx` 0, fresh effects the next Match |
| Arrow | -105.5 degrees against an independent bearing of -106 on Shard 4's Outer ground, and radially inward on CrownOuter and CrownInner (45 and 135 against 45 and 135), with the player placed by a temporary teleport |
| Beacons, every one of the 25 Section and connector models forced into Warning locally | 25 highlights and 25 beacons, each base on exposed floor of its own model (CrownOuter r = 26 on the visible ring, CrownInner r = 19 on the terrace, the Dais at its centre), none left over after clearing |
| A screenshot at 16 s left | The line, the alert, the beacon and the highlight all render |

**Not yet exercised live:** a real Match through Layers 2 to 4 with a player standing on them, the `LAST GROUND` and failsafe lines (specs only), a Hub viewer's line in a Running Match, a stalled-server shift on screen, a client joining mid-Warning, and how it reads on a phone or with a controller. The Highlight count is at most 25 here and Roblox renders 31, which holds for any one layer. How it looks and feels is for the developer to judge, and the art and audio belong to Ticket 26.

**State after step 6.** Edit mode shows the map intact (15 Sections, 12 connectors), `ServerStorage` holds only `Tests`, `Tools` and `MatchMapGreybox`, no `TEMPORARY` instance or text anywhere (grepped), `startingKit = "Reference"`, `maxSeconds = 600`, and with the Reference Kit the client builds an empty effects folder and runs nothing per frame. The `.rbxl` snapshot is not re-saved and no checkpoint is published.

**Step 7, telemetry (done 2026-11-11, 663 specs: 636 + 27, 0 failed).** Log lines only: no gameplay, attribute or remote changed, and with the Reference Kit nothing runs.

What was built:

- **`Collapse` gained an optional `observe` hook** (the Match's list of Alive players: id, position, seconds since damage, Stunned/Frozen/Rooted, plus full-Kit and Rank 2 counts). Without it every new field and line is absent. It runs in a protected call, before the event changes the world, and the post-event telemetry is guarded too, so a telemetry failure is counted (`telemetryFailures`) and never fails or retries an event.
- **Which ground a player stands on** is a ray down from 1 stud above their root, against the models still standing (`Collapse:_standingOn`).
- **Lines.** `CollapseStart` adds `players`, `schedule`, `layer4` and `voidKillY`. `CollapseWarn` and `CollapseRemoved` add `alive` and `aliveOnLayer`, and Warn adds `fullKits` and `rank2Players`. The loot counters were already there from step 3. `CollapseTick` (10 s and 5 s, from the pure `CollapseRules.ticksDue`) gives the Alive count, `onWarning` and their ids. `CollapseExposed` gives one row per player on the ground being removed: section, `sinceDamage`, `status` and `warned` (`yes`, `later` or `unknown`, from the 10 s snapshot). `QuietSpan` comes from the step 1 rules, polled by the Match loop, with Section occupancy counts. `CollapseSummary` is one line at Match end.
- **`Session`** gained `setDamageListener`, called for any damage to a player (`pvp` only between two players, a fully absorbed hit included). `CrownOpened` gains `fullKits` and `rank2Players`.
- **`Match`** owns the QuietSpan state, the PvP damage by phase, eliminations by `phase_cause`, Void counts (credited, self, Sentry, in a settle window), Kit relocations and consumptions, and deferred verdicts. It adds the `crown` checkpoint. The Collapse adds the Warning, removal and `dais` checkpoints, the summed loot and its own health counters.

Decisions taken while building, for review:

| Decision | Why |
|---|---|
| `CollapseExposed` leaves out the distance to the parent boundary and Dash readiness | The ticket says "if cheap" and "only if Movement already exposes it". The registry's boxes are coarse and `Movement` doesn't expose a charge count |
| The first Stunned, Frozen or Rooted attribute on the character is the `status` | One word is enough for a sample. `Rooted` isn't set by anything yet |
| Any damage to a player is "recent damage", but only player-versus-player damage is QuietSpan activity and counts as PvP | PvE and Sentry damage don't count as activity, as the ticket says |
| A fully absorbed hit counts as PvP activity, with 0 Health lost | The attack still happened. PvP damage by phase sums Health lost |
| `daisAlive` is everyone not standing on Layer 4's ground at its removal, and the Dais checkpoint and progression snapshot come from the same observation | The removal's Alive-entering count, taken before the ground goes |
| `CollapseTick` is skipped for a removal that is already due | The removal applies first, so a tick with no time left would mean nothing. A stall gives both ticks at once with their real `remaining` |
| The summary prints every loot counter the removals returned, plus `kitsRelocated` and `kitsConsumedAtDrop`, rather than a fixed list | Nothing is lost if a hook adds a counter. Whole-Kit drops swept from the ground are `pickups.kits` |
| `health.deferred` counts verdicts the settle window pushed back, scheduled by the Match | The other health counters come from the Collapse |

**Specs (27 new).** `CollapseRulesSpec` (3): `ticksDue` once each in order, a stall, and a shift. `SessionSpec` (3): the extended `CrownOpened`, the listener's pvp, absorbed and non-player cases, and a failing listener. `CollapseSpec` (12, on the fixture, with its own observe list): no hook means no new field, `CollapseStart`, Alive and on-layer counts, ticks, a stalled tick, `CollapseExposed` with `yes`, `later` and `unknown`, a failing hook, the summary (checkpoints, loot, delay, shifts, stale), a clean second summary, and occupancy. `MatchSpec` (9): the listener is installed only with a Collapse, the observe hook, QuietSpan (45 s, then 30 s, PvE ignored, PvP restarts it), none with one Alive, before GO or with no Collapse, `CrownOpened` and its checkpoint, the summary, Void counts, the Dais part, and no summary after a failed start.

**Live check on the real Rift Crown (accelerated schedule, solo, then reverted).** `CollapseStart` (`players=1 schedule=45/80/115/150 layer4=true voidKillY=-40`), `CrownOpened ... fullKits=0 rank2Players=0`, `CollapseWarn` at t=30 (`alive=1 aliveOnLayer=1`), `CollapseTick` at remaining 10.00 and 4.98 with the player's id, `CollapseRemoved` at 45.00 with `aliveOnLayer=1` and 12 Caches, `CollapseExposed` for the player on `Shard2Outer` (`warned=yes`), the `Eliminated` line with `phase=collapse sinceRemoval=1.00 settle=yes`, and `CollapseSummary` (`checkpoints=crown:1,warn1:1,remove1:1`, `elims.collapse_Void=1`, `voids.self=1`, `voids.settle=1`, every health counter 0). **Not yet exercised live:** `QuietSpan` (needs two Alive), `CollapseExposed` for a player who arrived later, the Dais fields, a real PvP count and `kitsRelocated`. They are specced, and step 9's two-client matrix covers them.

**State after step 7.** Tuning is back to the committed values (checked by loading a clone: `Reference`, 600, 60, 210, 30 and 90), the map is intact (15 Sections, 12 connectors), no holding folder and no temporary instance remain, and no `TEMPORARY` text is left outside the specs' own fixture names. The `.rbxl` snapshot is not re-saved and no checkpoint is published.

**Step 8, the structural checks (done 2026-11-11, 690 specs: 663 + 26 specs + the structural step itself, 0 failed).** Read-only, and nothing in the map is moved to check it.

What was built:

- **`CollapseChecks`** (server, new). `evaluate(data)` is pure and takes plain data. `read(collapse, opts)` gathers the data from the Collapse's registry and the world, and `run` does both. A problem is `"<check number>: <text>"`, sorted and with a break that several chains meet said once. Checks 1, 2 and 4 to 8 are on the data. Check 9 passes `MapLayout.validate`'s problems through when `opts.mapProblems` is given.
- **Check 3, Dais isolation, without a removal.** It never reparents anything. It treats every Layer 1 to 4 Section and connector as gone and then asks four questions of what is left:
  1. **Leftovers.** Does any `BasePart` in the workspace within 200 studs of the Dais (flat distance, edge to centre) still have `CanCollide` or `CanQuery` true, and belong neither to a removed model, the Dais, a character nor a runtime folder? A part with both off is the exempt presentation part.
  2. **Top.** Is the Dais top flat? Rays down at its centre and two rings (0.5 and 0.9 of the radius, 8 azimuths) must each hit an upward face (normal Y at least 0.99) within 0.1 stud of the top.
  3. **Ground and faces.** Is there no ground beyond the rim? Rays down at 16 azimuths and 6 distances (1.5 to 100 studs out), against everything except the removed models, must hit nothing, Terrain included. Horizontal rays at the top, the middle and the foot must meet the Dais first.
  4. **Self-contained.** Is every Dais part anchored, and nothing joined by a weld or joint to a part outside it?
- **Where it runs.** `RunAll.run()` starts a full run with it (one passing step, or one failure line for each problem), and `RunAll.structuralProblems()` is public. `Collapse:assertIntact` adds it when its deps carry `structure = {}`, which `Match` passes, so it runs at each Match's start inside `prepare` and the start's own assertion, and a failure refuses the Match as any integrity failure does. A spec fixture leaves it out. Before a `.rbxl` snapshot, call `RunAll.structuralProblems()` in Edit mode (through a clone, as the quirks say), which is also check 10.
- **Check 10.** It is the same run, in Edit mode after a Play test that consumed Sections. Play changes don't persist, so what it proves is that the authored map is whole.

Decisions taken while building, for review:

| Decision | Why |
|---|---|
| "Around the Dais" is 200 studs, measured on the ground plane from the Dais's centre to the nearest edge of the part | The map is about 260 across, so this reaches its rim with room to spare. The Arena and Hub are 700 or more studs away |
| Anything that is not in a removed model and has `CanCollide` or `CanQuery` true counts, wherever it sits in the workspace | A stray part outside `MatchMap` still blocks a player's body even when the game's queries ignore it. Runtime folders (Pickups, Caches, Sentries, `CollapseTelegraph`), characters, Terrain and the Camera are left out |
| Terrain is only checked by the rim rays, not by reading its voxels | The rays find terrain under or beside the Dais. A terrain-free map is the plan, and a voxel scan was not worth its cost |
| The Dais is a model of any shape, and its size comes from its world box | The real Dais is one cylinder stood on end (`GetBoundingBox` follows its pivot, as the step 5 quirk says), so the box's corners are taken into the world |
| Ground beyond the rim is a failure even if it is part of the Dais model | A ledge on the Dais is a landing place. Only a hit on the Dais at the foot, middle and top counts as a face |
| Some faults are named twice (check 3 leftovers and the face or ground rays, and checks 5 to 8 in `assertIntact`'s own list) | Each says what it saw, and the lists are short. Merging them would make `assertIntact` depend on the new module's order |
| Schedule checks (1 and 4) read `Tuning.collapse.rows` unless `opts.rows` is given | The checks describe the map and the committed schedule, not a Match's temporary one |

**Specs (26 new).** `CollapseChecksSpec`: 13 on plain data (a sound map, each of checks 1, 2, 4, 5 to 8 and the 3 and 9 passthrough, Layer 4 omitted, the sorting) and 13 on a fixture (`TEMPORARYChecksFixture`, far from the real map, destroyed in every case): sound, read-only, a removed layer's geometry ignored, a stray collidable part and both ways to be exempt, a far part ignored, a bump on the top, a ledge and a wall, an unanchored part and an outside weld, a held model, an unregistered one, a stale attribute and a moved one, the schedule, check 9, `assertIntact` with and without the option, and the live Rift Crown through `RunAll.structuralProblems`.

**Live checks on the real Rift Crown.** All ran in Edit mode with clones. The real map has 0 problems in about 9 ms. With temporary faults injected and then removed, each was found: a collidable part at r=40 (check 3, twice: leftover and blocked face), the same part with `CanCollide` and `CanQuery` off (none), a 1-stud bump on the Dais (not flat, y 6.00 against 5.00), a 1-wide wall (leftover, ground and blocked faces) and a stale `CollapseState` (check 7). In Play, `Collapse:prepare` with the option on the real map answered `ok=true` in 2.6 ms. After stopping Play, the check 10 run read 0 problems.

**Bug found by step 9a, fixed (2026-11-11).** The first live Match refused to start its Collapse (`CollapseResetFail structure check 3: ground remains round the Dais ... Workspace.Caches.CacheCrown1.Body ... Workspace.Sentries.Sentry1.Body`). A Match sets up its Caches and Sentries before `prepare` runs, and the rim rays (ground and faces) counted their bodies as ground. `RUNTIME_NAMES` was applied to the part scan only. The rim rays now also exclude the runtime folders (`Caches`, `Sentries`, `Pickups`, `CollapseTelegraph`) and player characters, Terrain stays in, and a spec places runtime objects beside the Dais. The Edit-mode runs and the Play `prepare` above could not show it, since none had runtime objects. Confirmed live afterwards with 21 Caches and 3 Sentries standing: 0 problems and `CollapseStart` logged. So the "Live checks" above prove the checks on a bare map, and step 9a's Matches are what prove the Match-start path. `CollapseChecksSpec` is 27 specs (the 26 plus this one), `RunAll` 691.

**State after step 8.** The map is intact, `ServerStorage` holds only `Tests`, `Tools` and `MatchMapGreybox`, no `TEMPORARY` instance exists, `startingKit = "Reference"`, and the `.rbxl` snapshot is not re-saved and no checkpoint is published.

**Step 9a, S1 solo on the real map and the reset proof (done 2026-11-11, 691 specs, 0 failed).** Settled with the developer: step 9 is split, 9a (S1 rows 1 to 15 and the reset proof, solo) now and 9b (S2, latency at 0, 100 and 250 ms, and the Tidal gate, which need a Server & Clients session) later, each with its own record and commit. S1 ran on the ticket's test-only schedule (removals 60 / 100 / 140 / 180, `warnSeconds` 15, `crownOpenSeconds` 0, `firstWarnGuardSeconds` 0, `maxSeconds` 300, Empty Kit), swapped into `Tuning` by the main session with marked, exactly reversible edits, then reverted: `Reference`, 60, 600, 210 / 285 / 360 / 435, 30, 90, no `TEMPORARY` text left, `RunAll` 691 green. `studio-tester` ran the matrix in three batches, and the main session reviewed each table. The first run found the Dais-check bug recorded under step 8 above, and its whole matrix was rerun after the fix.

| Row | Result | Evidence |
|---|---|---|
| 1 Lifecycle | PASS | Warning at 45 / 85 / 125 / 165 and Removed at 60.02 / 100.04 / 140.08 / 180.10, each once and in order, a whole layer in one step (7, 9, 7 and 1 models, the Dash markers never touched). `CollapseLayer` 1, 2, 3, 4, then 0, `CollapseWarning` false, true, false around each removal, the attributes absent before GO and cleared at the end. The effective times drift +0.004 to +0.08 s from the accumulated `shift`, matching the `CollapseWarn shift=` lines |
| 2 Removed geometry | PASS | For one Section of each layer at T-0.2, T+0.1 and T+0.5: before T it is in its home, hit by the ray and inside the box. From T it is in the holding folder, the ray misses and the box is empty |
| 3 Stay on Layer 1 | PASS | `CollapseExposed ... warned=yes`, `Eliminated cause=Void phase=collapse sinceRemoval=1.32 settle=yes`, `MatchEnd NoSurvivors` at 61.5 s (removal plus the 1.5 s settle, not earlier) |
| 4 Leave in time | PASS for 4a and 4b, 4c recorded | 4a: walk from r 91.9 at 10 s remaining, 15 studs in 1.22 s, survived. 4b: from r 61.3 at 5 s remaining to the terrace in 2.9 s, survived. 4c (about 1 s, recorded only): started at r 34.2 and was about half a stud onto the terrace at the removal, survived narrowly |
| 5 Fight while Warning | PASS | At 46 to 55 s on Layer 1: 13 forged Basic Attacks broke `CacheHome1A` at 51.27 s, the Pickup was collected, a Dash and the new Ability's Cast were accepted, all before T |
| 6 Pickup at the boundary | PASS, with a caveat | At all four boundaries a Pickup spawned at T-0.25 was collected and one spawned at T-0.02 was consumed by the removal (7 of 7). A Kit too full to take a Pickup (P2) stayed loose and was swept at T. The 0.1 s collect tick means a spawn 20 ms before T lands its tick before T about 20% of the time, so no trial showed collection winning. That is the expected outcome, not a gap in the ordering. The ordering is also covered by the specs |
| 7 Cache break at the boundary | PASS | Both Home Caches at 3 health. A hit at T-0.149 broke A (`CacheBroken`, one drop, which the removal then consumed). A hit at T+0.116 found B gone: no break, no loot, no error |
| 8 Sentry Telegraph across the removal | PASS | `SentryTelegraphEndsAt` set at T-0.383 and ending at T+0.117 (a second Sentry's ended at T+0.332). `Sentries` went 3 to 0 at 140.079. No damage, `Kill` or error after it (HP 100 throughout), `stale=0` |
| 9 Layer 3 to the Final Fight | PASS | No Sentry, Crown Caches still 3 and unbroken from 100 to 180, `CrownInner` Normal until its Warning at 165, and the status line points at Layer 4 ("Inner Crown consumed in 0:40") |
| 10 Layer 4 to the Dais | PASS | Only `CrownDais` left in `Sections`, Caches 0, Sentries 0, the line reads "LAST GROUND" with no countdown, the player alive on the Dais. A solo Match ends `Timeout winner=none`, with `dais.entered=1 dais.seconds=119.93 dais.timeout=true` |
| 11 Failsafe | PASS | A one-row schedule with `maxSeconds` 180: the notice first appeared at t=120.02 ("LAST GROUND · match ends in 1:00"), counted ceil of the time to GO+180, was never `0` or `0:00`, and the Match ended `Timeout winner=none alive=1` at 180.2 s. After the only removal `CollapseLayer` read 0 and no Collapse countdown showed |
| 12 Countdown | PASS | The text equals ceil(remaining) every second (never `0` or `0:00`), each change 0.03 to 0.1 s after the crossing (one 15 fps frame), no drift over a Warning, and the next layer's countdown 0.1 s after each removal. The tier change, observed on Warning ground: at 10 s remaining `MaxTextSize` 20 to 22, at 5 s to 26 and 24. Pulse crossings 0.41, 1.00 and 2.13 Hz against 0.5, 1 and 2 |
| 13 Presentation state | PASS | `CollapseTelegraph` children 0, then 14, 18, 14 and 2 (two for each model) at the four Warnings, each cleared within 0.1 s of its removal, none left after Layer 4. The alert and arrow showed only while the player stood on Warning ground. The announcement showed about 4 s with the right layer name |
| 14 Reset proof | PASS | Match 1 consumed all four layers, broke Caches, and ended; before the reset the holding folder held 24, at the next Countdown 0 with 15 Sections (none with `CollapseState`), 12 connectors and 29 markers. Match 2 passed the pre-start check and started from Layer 1 with 21 unbroken Caches, 3 Sentries, no stale Pickups or attributes, and `stale=0 failures=0 resetFailures=0` in every `CollapseSummary` read (Matches 3, 4 and 6, and 1 in the failsafe run). Ended mid-Warning: 7 models were in `Warning` at 45.5 s, the player was killed at 50.0 s, and on the first frame after `MatchEnd` no model had a state and all four Workspace attributes were gone. The next Match started cleanly |
| 15 Six-roster sanity | PASS | 200 seeds of `MatchRules.assignSpawns` gave 6 distinct pads for six players, and a pair was opposite in every seed. Six rigs on a ring at r 7.5 on the Dais (radius 14, top y 5) each had the Dais under them (normal Y 1.000), 7.5 studs apart. The schedule and `validate` take no player count |

Not required by S1, and not proven here: the Match 7 `CollapseSummary` (Play was stopped before it ended), and the alert's own background pulse in the final tier (the alert hides at 3.5 s once the player leaves, so the Highlight's pulse was measured instead).

Quirks found are recorded in [`studio-mcp-quirks.md`](testing/studio-mcp-quirks.md). Decisions and observations for the developer:

| Observation | Note |
|---|---|
| The Crown Sentries kill a player standing on the Dais before Layer 3 falls (seen twice, at about 15 s and 43.8 s) | This is the settled finding 1 of Ticket 21 (the Dais stays under all three Sentries until Layer 3 falls). A blind spot on the terrace, found by a line-of-sight check from each Sentry's fire point, took no shot for 26 s |
| A solo Match never ends in a `Winner` | One Alive at Match end by Timeout is `Timeout winner=none`, as the rules say. It makes solo Matches useful only for the Collapse itself |
| `MatchGoAt` is set when Running begins, about 3 s before GO, and the Collapse attributes appear at GO + 0.02 | As built (the arrival hold). Nothing reads the Collapse attributes before GO |

**State after step 9a.** `Tuning` is at the committed values, Edit mode shows the map intact (15 Sections, 12 connectors, 0 structural problems), `ServerStorage` holds `Tests`, `Tools` and `MatchMapGreybox`, `Workspace` has 5 children and no attributes, and no `TEMPORARY` instance or text remains outside the specs' fixture names. The `.rbxl` snapshot is not re-saved and no checkpoint is published. **Still to do in step 9:** 9b (S2 rows 16 to 23, the latency settle-margin test and the Tidal gate), which needs your Server & Clients session.

**Step 9b, Tidal gate plan (settled with the developer 2026-11-11, before running).** The gate runs solo in Play now, and S2 and the latency tests wait for the developer's Server & Clients session. The victim is the real solo player, because a player's own client applies Knockback. The attacker is a server cast from a point on the opposite side of the victim, so the push points outward. A server cast is always a tap, so charged hits need a marked, exactly reversible `TEMPORARY` option on `Casts.castFromServer`, removed afterwards. Surfaces: flat ground and the real Dais (a cylinder of radius 14, top at y 5). Committed Tidal values, no retuning: tap `pushStrength` 25 and full 40, each for 0.2 s, so about 5 and 8 studs expected. Cells: `hitRadius` 0, 3, 6, 9, 12 and near the rim, tap and full, 5 trials each, with the victim stationary, with input forward and with input away from the push. Then Dash-after-hit at 0, 0.1, 0.2 and 0.3 s for `hitRadius` 6 and below, and a Stone Guard check first (a Shield must not stop Knockback, or that is a bug to fix before reading the result). Verdict by the criterion above, with the Dash success rate recorded as evidence.

**Step 9b, S2, latency, the settle value and the Tidal gate (done 2026-11-11, 691 specs, 0 failed).** `studio-tester` ran everything and the main session reviewed each table. Everything ran under the same marked, exactly reversible test schedule as 9a (reverted afterwards: `Reference`, 210 / 285 / 360 / 435, 30, 90, 60, 600, no `TEMPORARY` text or instance left, 15 Sections, no Workspace attributes). **`Tuning.collapse.settleSeconds` is now 1.75** (see the latency result), and the specs that asserted 1.5 now read it from `Tuning`.

**S2, two clients (rows 16 to 23, 14 Matches).**

| Row | Result | Evidence |
|---|---|---|
| 16 Fight on Warning terrain | PASS | 47 to 60 s on Layer 1: 13 forged Basic Attacks (a Cache broke and its Pickup was collected), a Dash, a server Tidal Bolt (12 damage, knockback), all accepted, no rejection on either client |
| 17 Two players fall on different Sections | PASS | Both `Eliminated cause=Void phase=collapse`, `MatchEnd cause=NoSurvivors` at the removal plus the settle, in three Matches |
| 18 One falls, one safe | PASS | `MatchEnd cause=Winner` at the removal plus 1.5 s (the value then), 0.47 s after the fall, never at the fall |
| 19 Combat death and a fall in the same window | PASS | A death 0.04 s before the removal and one 0.30 s after: both `NoSurvivors`, whatever the order. See the observation below for a death 0.3 s before |
| 20 Displacement just before removal | PASS | A bolt landing at T-0.18 pushed one player inward onto live ground (survived) and left the other on Layer 1 (fell as `Void`). With a real roster attacker: `void=Knockback by=player:-2 ability=TidalBolt chargeBand=Tap hitRadius=100.0 hitDist=5.0`, then `Kill`. A server dummy gives no credit, by design. Hook not tested: no Kit |
| 21 Whole-Kit drop on vanished ground | PASS for Branch 1 | `KitDropRelocated from=Shard1Outer hops=1 result=relocated to=Shard1Inner`, and a floor raycast at the Pickup hit `Shard1Inner` at y 0. **Branch 2 is not reachable in a live Match:** every `CollapseParent` chain ends in the Dais, which is never removed, so the consumed branch is covered by the specs only |
| 22 Dais Last Stand, 2 players | PASS | Layer 4 removed at 180.07 s with both alive, "LAST GROUND" 0.1 to 0.2 s later, no countdown, one player killed the other on the Dais: `MatchEnd cause=Winner` at 189.9 s, not Timeout |
| 23 Spectator | PASS, limited | `CollapseAlert.Visible=false` while spectating, the status line and the world telegraphs present (14 telegraph children). The arrow is a child of the alert, so it is hidden with it. A spectator watching later Warnings needs 3 or more players |

**Latency (blocking measurements, 0, 100 and 250 ms, 500 ms as stress evidence, both clients as observers, 11 Matches).** The lag was `IncomingReplicationLag`, confirmed effective (250 ms gave a receipt lag of 0.27 to 0.30 s) and restored to 0. Times are after the applied removal.

| Lag | Client receipt = geometry gone (median) | Physical fall start (median) | Server `sinceRemoval` (min / median / max) |
|---|---|---|---|
| 0 ms | 0.047 | 0.089 | 0.98 / 1.04 / 1.25 |
| 100 ms | 0.138 | 0.187 | 0.97 / 1.00 / 1.07 |
| 250 ms | 0.291 | 0.337 | 1.22 / 1.27 / 1.37 |
| 500 ms (stress) | 0.522 | 0.564 | 1.38 / 1.43 / 1.47 |

- **The pass rule failed at the initial 1.5.** The worst Void offset across the blocking settings was 1.37 s (and 1.30 s in an S2 idle Match at 0 ms), so the worst plus 0.25 was 1.62, above 1.5. **The chosen value is 1.75**, the smallest 0.25 step that passes with the 500 ms stress case too (1.47 plus 0.25 is 1.72).
- **Rerun at 1.75 (5 Matches, every `CollapseStart` read `settleSeconds=1.75`).** The worst was 0.92 s at 0 ms and 1.32 s at 250 ms, so the rule passes (worst 1.50 or less). The 500 ms stress Match read 1.55 and misses the rule by 0.05 s, which is stress evidence only. Every `MatchEnd` landed 1.76 to 1.78 s after the removal and after the last Void elimination (margins 0.2 to 0.9 s).
- **Why the offset grows with lag:** a removal received late means a late fall to the kill plane (about 0.7 s of free fall), and the server's void check runs on 0.25 s ticks (`loopSeconds`).
- **Visual fairness:** nothing looked unfair. The geometry disappeared at the same moment the client received the removal, the fall began about 0.05 s later, and the server decided about 1 s after that, so no player stood on ground the server had already removed. The countdown read `0:01` at the removal, never `0:00` and never a number after it, and switched to the next layer 0.15 s (0 ms) to 0.35 s (250 ms) later.
- **Limits:** `IncomingReplicationLag` understates what a real laggy player sees of other players, and outgoing lag and a client's own position reaching the server late were not tested. Every sample stood at the same fall height (root y 3.2).

**The Tidal gate (measurement, performed and recorded).** Committed Tidal values, no retuning. The victim was the real player, and the attacker was a server cast through a marked, removed test option.

| Evidence | Result |
|---|---|
| Push distance | Tap about 5.4 studs (5.2 to 5.6), full Charge about 8.5 (8.2 to 8.9, 9 from rim effects), against 25 and 40 studs/s for 0.2 s. Flat ground gives the same (5.3 to 5.5 and 8.5 to 8.8) |
| Damage | 12 on a tap and 15 on a full Charge |
| Dais | A cylinder of radius 14 with its top at y 5. A root stays on it up to about r 14.5 |
| Full Charge, stationary victim, ring-outs | `hitRadius` 0 and 3: 0/5 each. `hitRadius` 6: 4/5 in one run and 0/5 in the Dash run's control (4 of 10). `hitRadius` 9, 12 and 13: 5/5. A tap rings out only from `hitRadius` 12 (5/5) |
| With and without input (simulated Move) | Moving toward the centre: 5/5 survived at about 3 and 6. Moving away: 4/4 rang out at both |
| Dash after the hit (real LeftShift, delay measured on the client) | Delays 0.02 to 0.33 s: 16 of 16 survived, ending 5 to 11 studs toward the centre. Delays of 0.37 s and more: 3 of 6 survived, decided by whether the push had already carried the root past about r 15.7 |
| Stone Guard | A Rank 1 Shield absorbed all 15 damage and the push was unchanged (9.74 studs against 8.76 unshielded). Not a bug, and by the code the Shield only changes the damage |
| Doc contradiction | Settled: 5 studs per tap, 8 per full Charge. The Ticket 21 note stands |

**Reading against the criterion.** A single charged Tidal hit at `hitRadius` 6 or less does **not** reliably ring out a stationary victim (0 of 5 at 0 and 3, 4 of 10 at 6, with the push ending right at the rim), and a Dash 0 to 0.33 s after the hit always saves the victim, but surviving did not need a Dash (the control survived without one). The Dais is an edge-fight arena rather than a centre lottery: from `hitRadius` 9 outward, one full Charge rings out every time. **Decision (developer, 2026-11-11): Layer 4 ships enabled**, on the evidence of sufficient counterplay near the centre of the Dais, with `collapseSettleSeconds` kept at 1.75. The limitations stay on record: the movement cells used simulated `Humanoid:Move`, not real keys; real-key Dash timing at 15 fps jitters by about 0.1 s; the solo Tidal runs had no Void, so a ring-out landed on the Terrace 5 studs below; and the **500 ms stress case misses the pass rule by 0.05 s** (worst `sinceRemoval` 1.55 against a limit of 1.50), a known limitation that was never a blocking setting. Caveats: the movement cells used simulated `Humanoid:Move`, real-key timing at 15 fps jitters by about 0.1 s, and the solo runs had no Void, so a ring-out landed on the Terrace 5 studs below.

**Observations for the developer.**

| Observation | Note |
|---|---|
| A combat death 0.3 s before a removal ended the Match as `Winner` at T-0.05, while the survivor stood on ground about to vanish. A death 0.04 s before gave `NoSurvivors` | Explained and **corrected by the developer's decision**, see the boundary rule below. Rows 18 and 19 above describe the behaviour before the correction (a death 0.04 s before the removal gave `NoSurvivors`) |
| With 2 players the Match ends right after the first death, so a spectator sees the Collapse for 0.3 s | A real spectator check needs 3 or more players |
| Branch 2 of row 21 (no live ancestor) cannot happen on the committed map | Covered by the specs, not live |
| Ticket 28 (slow motion and a Win-cam on the final kill) was proposed during this step | Written as a staged idea, not built |

New quirks are in [`studio-mcp-quirks.md`](testing/studio-mcp-quirks.md).

**Step 9b boundary correction: a kill before the removal is final (decided and done 2026-11-11, 699 specs, 0 failed).** The S2 observation above came from two clocks that did not line up. `MatchRules.checkEnd` made a pending end at `death + 0.25 s`, and `Collapse.complete` opened the settle window at the **applied** removal. `Match.schedulePendingEnd` re-read the live `settleUntil`, so a removal applied inside that 0.25 s grace pushed an already-pending verdict out to `removal + 1.75 s`. A death 0.3 s before the removal resolved first and won, and a death 0.04 s before it was held and became `NoSurvivors` for a survivor on ground that vanished. The cliff was at a death 0.25 s before the removal. It was not event ordering (`eliminate` calls `tick` synchronously, and `tick` applies due removals first) and not the 1.75 s length. **The developer's preference, approved as Option B:** a player who eliminates the final opponent while still alive wins, even if a removal is imminent, and genuinely simultaneous eliminations still draw.

- **The change (narrow).** `MatchRules.checkEnd` no longer re-reads the settle window once an end is pending: `pendingEndAt` is fixed at creation as `resolveAt(death + endResolutionSeconds, settle)`, with a `pendingHeld` flag for the `deferred` telemetry. `Match.schedulePendingEnd` uses that fixed deadline. Nothing else moved: the 0.25 s grace, the 1.75 s window for Void and post-removal kills, and removal-first ordering at the same timestamp are as before, and no predicted-safety check exists.
- **Specs.** 691 became 699. Two specs that asserted the extension were flipped (`MatchRulesSpec` "a window that opens after the grace began does not move the resolution out" and the `MatchSpec` removal-after-grace case). Added, with exact binary-fraction times: deaths well before, just before, at and after the removal; a committed attack that lands inside the grace (with the removal applied before or between the two deaths); two Void falls at different times after the removal held to the window; a pending end created inside an open window keeps respecting it and a later window changes nothing; and a `MatchSpec` integration case with an open window at the final death. A **mutation check** (the old `resolveAt` line put back temporarily) failed 6 specs, then the exact revert gave 699 passed, 0 failed.
- **Live check (S2 row 19 rerun and the boundary scenarios, 2 clients, the test schedule, `studio-tester`).** Both players idle on different Layer 1 Sections, and a server write of `Humanoid.Health = 0` at an absolute server time stood in for a combat kill (the same path as `Combat.applyHit`'s lethal damage). Times are seconds from `CollapseRemovesAt` (T); the applied removal was T+0.00 to +0.03.

| Scenario | Actual | Expected | Result |
|---|---|---|---|
| B, kill just before the removal | P2 died at T-0.069, the removal applied 0.09 s later. `MatchEnd cause=Winner winner=P1` at T+0.216 (kill + 0.285, with the 0.01 s delay and a frame). The survivor stood on Layer 1 and had started to fall (root Y 0.40, standing 3.2) and was never Void-eliminated | Winner at the death + 0.25, not removal + 1.75 | PASS |
| A, kill well before | P2 died at T-0.483. `Winner` at T-0.199, before any removal was applied | Winner at death + 0.25 | PASS |
| C, kill after the removal, inside the window | P2 died at T+0.316 (`sinceRemoval=0.32 settle=yes`). P1 was Void-eliminated at `sinceRemoval=0.95`. `NoSurvivors` at T+1.768 | Held to removal + 1.75, P1 falls, draw | PASS |
| D, two kills inside the grace, before the removal | P2 at T-0.286, P1 at T-0.185. `NoSurvivors` at T-0.001 (first death + 0.25), no removal applied | NoSurvivors at the first death + 0.25 | PASS |
| E, grace across the removal | P2 at T-0.084, P1 at T+0.067 (after the removal). `NoSurvivors` at T+0.201 (first death + 0.25), not held to removal + 1.75 | NoSurvivors at about removal + 0.15 | PASS |
| F, two Void falls, no kills | Both fell at `sinceRemoval=1.02`. `NoSurvivors` at T+1.783, 0.8 s after both, no Winner for the first to fall | NoSurvivors at about removal + 1.75, after the later fall | PASS. The two values were equal, where I had guessed they would differ |

**Observations from the live check.** `health.deferred` was 1 only in C and F, where a Void fall came after the removal, and 0 elsewhere (every `CollapseSummary` read `failures=0 invalid=0 resetFailures=0 stale=0`). In B the Match ended with the survivor already falling for 0.2 s, so a Winner can be seen in mid-air for the end banner. The server write stood in for a combat kill, and a real killer's credit path was not exercised in this rerun (S2 row 20 did exercise it). **S2 row 19 now reads:** a final kill before the removal is applied wins at the death + 0.25 s whatever the survivor's ground, and a final kill after it draws if the survivor falls inside the window. Branch notes: a kill by a Sentry, a Warden or an eliminated player's projectile follows the same path, since the rule does not use the killer.

**State after step 9b.** `Tuning` is at the committed values with `settleSeconds = 1.75`, `RunAll` is green at 699 (691 before the boundary correction), and the map is intact. The `.rbxl` snapshot is not re-saved and no checkpoint is published. **Still to do in step 10:** the cleanup, documentation and the final commit, and the shipping decision for Layer 4 from the Tidal gate.

**Step 10, cleanup, documentation and the final record (done 2026-11-11, 699 specs, 0 failed).** Documents corrected for the facts of 9b: the settle value 1.75 and the boundary rule in `rift-collapse-design.md` (the original wording kept and marked "Amended 2026-11-11"), the settle and `pendingHeld` description in `code-structure.md`, this ticket's Status header, settle table row, settle bullet and the two resolved "Deferred" items. `CONTEXT.md` needed no change: its Match, Rift Collapse, Section, Warning, Consumed and Last Stand entries already match, and the settle window and end grace are implementation terms that stay in the ticket and the design note, so no term was added. `studio-mcp-quirks.md` needed nothing new. Tickets 27 and 28 are untouched.

**Final verification (solo Play, then Edit).** `RunAll` on the Server: 699 passed, 0 failed. `RunAll.structuralProblems()`: 0 problems, with 15 Sections and 12 connectors, in both Play and Edit. A search of every instance name and script `Source` outside `ServerStorage.Tests` (Workspace, ServerStorage, ServerScriptService, ReplicatedStorage, the Starter containers, Lighting, ReplicatedFirst) for `TEMPORARY`: none. Edit-mode `Workspace` has 5 children (Arena, Hub, MatchMap, Terrain, Camera) and no attributes (in Play it also holds the runtime Pickups folder, the four dummies and the player). `Tuning` source reads `startingKit = "Reference"`, `maxSeconds = 600`, rows 210 / 285 / 360 / 435, `warnSeconds = 30`, `firstWarnGuardSeconds = 90`, `crownOpenSeconds = 60`, `settleSeconds = 1.75`. (A `require` of `Tuning` from Edit returned the 9a test values, the known stale-module quirk in `studio-mcp-quirks.md`. The source and the fresh Play read agree with the committed values.) Play was stopped.

**Completion table (the 13 blocking criteria).**

| # | Criterion | Result | Evidence |
|---|---|---|---|
| 1 | All automated specs green in Play | PASS | `RunAll` 699 passed, 0 failed, rerun in step 10 |
| 2 | Structural and integrity checks pass | PASS | Step 8 (`CollapseChecks`, 26 specs, live checks with injected faults found) and the step 10 run: 0 problems, 15 Sections, 12 connectors. The same check runs at every Match start in `prepare` |
| 3 | S1 real-map matrix passes | PASS | Step 9a, rows 1 to 15 |
| 4 | S2 two-client matrix passes | PASS | Step 9b, rows 16 to 23 (row 23 "limited", row 21 Branch 2 specs only, both recorded). Rows 18 and 19 were rerun after the boundary correction, scenarios A to F |
| 5 | Reset proven by starting a second Match | PASS | Step 9a row 14 (Matches 2, 3, 4 and 6) and the 14 Matches of 9b |
| 6 | Latency settle-margin test passes, after tuning | PASS | Step 9b latency table: initial 1.5 failed (worst 1.37 plus 0.25), set to 1.75, rerun worst 0.92 s at 0 ms and 1.32 s at 250 ms. The 500 ms stress miss (1.55 against 1.50) is a recorded, accepted limitation and not a blocking setting |
| 7 | Tidal gate performed and documented | PASS | Step 9b, "The Tidal gate" table, reading and limitations |
| 8 | Layer 4 decision follows the Tidal result | PASS | Developer decision 2026-11-11: Layer 4 ships enabled, on the recorded counterplay evidence. The committed schedule has the Layer 4 row at 435 |
| 9 | Warning and presentation state create and clean up | PASS | Step 9a row 13 (`CollapseTelegraph` children 0, 14, 18, 14, 2, each cleared within 0.1 s of its removal, none after Layer 4), row 14 (ended mid-Warning: no state or attributes on the first frame after `MatchEnd`), S2 row 23 and the step 6 specs |
| 10 | No stale Match or Collapse callback affects a later Match | PASS | `stale=0 failures=0 resetFailures=0` in every `CollapseSummary` read (9a rows 8 and 14, 9b boundary check), the second and later Matches starting cleanly, and the reset and stale-callback specs. Not read: the Match 7 summary in 9a (Play was stopped first), noted there |
| 11 | Temporary tuning, network settings and test aids restored | PASS | 9a and 9b state records (no `TEMPORARY` left, `Reference` kit and committed schedule, `IncomingReplicationLag` back to 0) and the step 10 search: none |
| 12 | Edit-mode authored Arena intact | PASS | Step 10: Edit-mode 15 Sections, 12 connectors, 0 problems, `Workspace` 5 children and no attributes (also after 9a and 9b) |
| 13 | Studio and MCP quirks documentation updated | PASS | `studio-mcp-quirks.md` carries the entries from steps 3 to 9b (the last ones for step 9b: edge ring-outs, `Humanoid:Move`, boundary kills, `CollapseWarn` arming, specs reading measured Tuning, server `Health = 0`, the mutation check). Step 10 found nothing new |

The audit found every criterion has recorded evidence. The names differ slightly from the ticket text (`collapseSettleSeconds` is `Tuning.collapse.settleSeconds`).

**Not done, and not mine to do.** The `.rbxl` snapshot is **not re-saved** and **no checkpoint is published**. Version 125 is still the latest published checkpoint. The developer re-saves the snapshot, publishes, and records the version in Checkpoints.


### 23. Rift Warden

**Status: specified 2026-11-09 and reviewed adversarially the same day, not built.** Every design point was grilled with the developer and settled. The reasoning, rejected options and edge cases are in [`rift-warden-design.md`](rift-warden-design.md) and [ADR-0008](adr/0008-pve-actors-are-kinematic-server-models.md). This entry is the build contract. Values carry the same four labels as Ticket 22: **settled initial value**, **tunable**, **measurement-dependent**, **test-only**.

**Targets and balance expectations are not pass criteria.** The 15 to 20 s fight, the 35 to 50 Health cost, the Swing-backstep watch item, chase speed, route-tax feel and visual smoothness produce tuning notes. Only the **Blocking completion criteria** decide completion.

#### Settled before building (2026-11-12)

Settled with the developer at the start of the build. Ticket 21's arena geometry is never modified to fit a Warden.

- **Slot offset: run the experiment.** Test +15 to +20 degrees on the real map, with +10 degrees as the fallback. Evaluate navigation, combat space, connector clearance, cover, Collapse behaviour and spawn fairness. Prefer the larger offset only if it demonstrably improves positioning with no new problem, and if several pass equally, prefer the smallest change. Record the measurements and the chosen offset.
- **Territory radius: measure first.** Derive it from the real Layer 2 platform, keeping the loot-safe lip margin (about 15). Use the largest valid radius in the 7 to 12 range that keeps the safety margins and the combat space, and validate the distances to lethal edges and connectors. If none is valid, stop and report the measurements and alternatives.
- **Delegation.** Main session: `WardenRules`, the `Wardens` runtime, the `Combat` veto and integration, `Match` integration, architecture and the final review. `studio-implementer`: the written specs (with explicit acceptance requirements) and cosmetic `WardenVisuals`. `studio-tester`: the Play matrix. One Studio driver at a time, and delegated work is reviewed before it is accepted.
- **Build order.** (1) `WardenRules` and specs, (2) combat hooks (gate, Warded and Resisted, the `Casts` players-only filter), (3) `Wardens` runtime, slots, markers and validators, (4) `Match` integration, (5) visuals and telemetry, (6) the Studio Play matrix and regression. Specs and integration checks run after each major stage, and work never continues past a failed blocking gate. Completed, tested stages are committed locally.
- **Stop and ask if:** the slot or territory geometry can't satisfy the specification, a combat integration changes approved gameplay, a dependency or architecture conflict expands scope, a fairness problem can't be resolved within the specification, or a decision would modify Ticket 21's geometry.
- **Assets.** Placeholders and existing sounds only. Readability over polish. No upload, push or publish without explicit approval. Studio stays in Edit mode when not testing.

#### Build record

**Step 1, `WardenRules` and its specs (done 2026-11-12, 764 specs, 0 failed).** `Tuning.wardens` holds every number from this ticket as a starting value. `WardenRules` (pure, `Shared/Rules`) holds population and Gate selection (`count`, `select`, only the injected stream drawn and only on a tie), the distinct reward deal, the four-phase FSM with the timestamp-driven action (`step`, `noteDamage`, `acquire`, `challenger`, `chooseAttack`, `slamCentre`), regeneration, the stage gate, locomotion maths (`goalFor`, `updateGoal`, `stepToward`, `turn`, `clampToTerritory`), no-progress, fault and step checks, and `validate` for the config. `WardenRulesSpec` (65 tests, written by `studio-implementer` from a written list, reviewed and re-run by the main session in `RunAll` on the Server in Play) covers S1 to S6, the inert case and the config. Decisions the ticket left open, taken here and recorded:

- **Retention radius is `Territory + AttackReach`**, the whole area from which an accepted hit can arrive, so a player the gate allows to damage the Warden is also one it can keep as a target. Acquire still uses `AggroRadius`. A target beyond retention counts as no target.
- **A Warden with no target re-acquires** by the aggro rule (`TargetSwitched`) before the 4 s hold runs out, and the hold only runs while nobody is acquirable.
- **The Slam cooldown counts from the Telegraph's start.** The Swing cycle (0.7 + 0.7 + 0.8 = 2.2 s) is the same as the ticket's.
- **A Warden that arrives home at full Health resets at once.** Otherwise it waits `regenDelaySeconds`, heals linearly over `regenSeconds`, and then resets, and an accepted hit stops it with no instant cleanse.

#### Summary

The **Rift Warden** is the Arena's first mobile PvE enemy: an optional, contested, **elite-tier (mini-boss-style) guardian** standing on an inner-Shard platform near the Safe spoke. It carries a visible Ability reward, fights with two Telegraphed melee attacks, stays on its small Territory, and drops one ordinary Rank 1 Ability Pickup when killed. It is not mandatory for any route, Kit or progression step. It is seasoning around the PvP Match, and it is not a normal monster, a boss, or the baseline monster population.

#### Scope

- `WardenRules` (pure): selection, reward deal, FSM, targeting, attack choice, locomotion maths, regeneration, no-progress, fault, stage gate.
- `Wardens` (server): runtime, registration, motion, attacks, reward, `finish`, `removeSection`, telemetry.
- Warden slot and pad markers, the static validators, and the slot-offset experiment.
- A target-level pre-damage veto and the Warded and Resisted result reasons.
- A players-only server cast filter and a ground-circle form for `castFromServer`, only if `Casts` lacks them.
- `Match` integration, `WardenVisuals`, `Feedback` additions, `Protocol` and `Tuning` entries, specs, the Studio matrix, `CONTEXT.md` terms.

#### Non-goals

- Regular monsters, an elite or Dais Warden, a boss, Match XP, Relics, Equipment, currency, permanent progression, cosmetics.
- Respawn, patrol, a leap, a ranged attack, any Warden Status, Root, Slow, Stun or player Knockback.
- A `MonsterService`, behaviour trees, `PathfindingService`, a generic enemy framework.
- Dynamic Warden scaling after GO, an independent despawn timer.
- Final art, audio and animation, camera shake, a reward icon panel, a nameplate, a minimap marker. **No upload without explicit approval.**
- Analytics backend, heatmaps, positional sampling.
- Changing the 24-drop Cache economy, `LootRules.validate`, or the shared Match `random` draw order.

#### Dependencies on Tickets 21 and 22

**Relies on (Ticket 21):** the three Warden slot markers on the inner Safe-spoke Shards (Layer 2), the pads and `ShardId`, Forward Caches, the Arrival Hold and `goAt`, the 24-drop economy, Crown Sentries, markers carrying `SectionId`.

**Ticket 23 adds to the Ticket 21 markers:** a `GateId` on each Warden slot and on each spawn pad, a `TerritoryRadius` on each slot, and the +15 to +20 degree offset experiment (the +10 degree position is the fallback).

**Relies on (Ticket 22):** `Collapse.sync(now)`, Section removal as one non-yielding step, restore before marker discovery, and the rule that a combat event accepted before the removal boundary resolves under the old world and one accepted at or after it under the new world.

**Requires from Ticket 22:** its object-cleanup step calls `Wardens.removeSection(sectionId)`, as it does for the Caches and Sentries it owns, and never destroys Warden instances itself.

**Build order.** After the Ticket 21 build, since the Warden needs the real Layer 2 platforms, markers, Gates and `goAt`. Tickets 22 and 23 can then land in either order. If Ticket 22 is not yet built, `removeSection` is built and tested alone. Nothing is faked. The real Collapse plus Warden Studio check becomes blocking once both implementations exist together.

#### Population and slot selection

Chosen once at Match setup from the roster, after `assignSpawns`. **Tunable initial population rule:**

| Match players | Wardens |
|---|---:|
| 1 (Studio solo only) | 1 |
| 2 | 1 |
| 3 to 4 | 2 |
| 5 to 6 | 3 |

Selection counts occupied pads per `GateId`. 1 Warden takes the least-occupied Gate. 2 Wardens take all but the least-occupied Gate. 3 take all. Ties use `wardenRandom`. No runtime geometry. Occupancies from the spread table: 2 players `{1,1,0}` takes the empty Gate. 3 players `{1,1,1}` takes any two. 4 players `{2,1,1}` takes the 2-player Gate plus one other. 5 and 6 take all three. Population is never changed after GO, and one Warden life per slot is final.

`wardenRandom` is a dedicated stream, injected (`deps.wardenRandom`, default `Random.new()`). It never draws from `Match`'s shared `random`, so the Cache deal and spawn assignment are unchanged. If a stable Match seed already exists, derive from it. Do not build seed infrastructure for this ticket.

#### Territory, aggro and targeting

| Rule | Value | Label |
|---|---|---|
| Home Territory | circle around the slot marker, never leaves it | settled |
| `TerritoryRadius` | geometry-derived, about 7 to 12 | measurement-dependent |
| `AggroRadius` | `Territory + 4` (never more), about 11 to 16 across the likely Territory sizes (about 16 at a 12 radius) | tunable, geometry-validated |
| Acquire | Running and past `goAt`, player Alive and valid, inside `AggroRadius`, clear authoritative line of sight (the Sentry sight helper) | settled |
| Retention | target validity and distance only, not line of sight | settled |
| Damage wake | any accepted player damage wakes it | settled |
| Stage gate | `distance(attacker, Home) <= TerritoryRadius + AttackReach` at acceptance | settled rule |
| `AttackReach` | 11 (Slam centre 6 plus radius 5) | settled initial value |
| Rejected hit | no damage, Status, Reaction, Threat or wake, plus a Warded result | settled |
| Threat window | 4 s of damage dealt | tunable |
| Switch | challenger at least 1.5 times the target's and at least a small absolute minimum (`threatMinDamage`), and 2 s since the last switch | tunable |
| Initial target | nearest valid player, tie by id order | settled |
| Disengage | 4 s with no valid target | tunable |
| Return | normal speed, targetable, damageable, re-engages on a valid player or accepted hit | settled |
| Regeneration | 3 s with no accepted damage after arrival, then to full over about 8 s, stopped at once by accepted damage | tunable |
| Full reset | Health full, Statuses and Threat cleared, target cleared, back to Idle | settled |

A committed attack never retargets. A kill elsewhere affects targeting only if the victim was the target.

The Warden is inert before `goAt`: no aggro, movement, attack or targeting. The phase is already Running during the Arrival Hold, so it must check `goAt` as Sentry targeting does. It must not rely on the Match phase alone.

#### Combat

**Identity:** a slow melee bruiser with one readable basic attack and one committed anti-kite area attack. Exactly two attacks.

| | Stone Swing | Rift Slam |
|---|---|---|
| Shape | flat frontal sector, reach 7, about 100 degrees | ground circle, radius 5, centre up to 6 from the Warden |
| Lock | facing at Telegraph start | centre at Telegraph start |
| Telegraph | 0.7 s, rooted | 1.2 s, rooted |
| Impact | one authoritative evaluation at `impactAt` | same |
| Damage | 12 | 22 |
| Recovery | about 0.7 s rooted, then about 0.8 s attack gap (movement allowed) | 1.2 s rooted |
| Cooldown | none beyond the cycle (about 2.2 s) | about 7 s |

All values are **settled initial values and tunable**. Attack choice: Swing if the target is inside Swing reach, Slam if outside Swing reach and inside Slam reach and ready, otherwise chase. No weighted AI. A hit shape hits every valid player inside it once per attack, never the Warden, Sentries, Caches, Pickups or map objects (an explicit players-only filter). No chaining. An impact resolves only while the Match is Running and past `goAt` (the existing hit gates), so a Warden never hurts anyone during the Arrival Hold or Results.

| Item | Value | Label |
|---|---|---|
| Health | 200 (range 180 to 260) | tunable |
| Damage to players | normal damage and Shield pipeline, no scaling by player count, Rank or time | settled |
| Evade | accepted Dash negates it exactly like normal hits, with the normal `Evaded` feedback | settled |
| Statuses and displacement against players | none, none | settled |
| Charging | ordinary Warden damage interrupts Charging only if existing damage already does | settled |
| Wet, Reactions, Arc Burst | work on the Warden normally | settled |
| Forced movement against it | immune (Tidal Knockback, Hook pull move nothing), damage and Statuses still apply | settled |
| Falling | cannot be displaced or fall, no Warden Void kill | settled |
| Kill Recovery | none | settled |
| Kill credit | `hit.attackerId` of the accepted lethal player hit (Last Hit), attribution only | settled |

A player killed by a Warden is eliminated with no killer and no Kill Recovery. Reuse the Sentry convention for the elimination cause. Warden damage to players is real combat damage but is kept out of PvP kill and damage numbers (as Sentries are in `SessionRules`) and is logged separately.

#### Locomotion

Anchored, kinematic and server-moved ([ADR-0008](adr/0008-pve-actors-are-kinematic-server-models.md)).

| Item | Value | Label |
|---|---|---|
| Chase speed | 12 studs/s (range 11 to 14) | tunable |
| Approach spacing | about 5 studs, an approach target and never a repulsion radius | tunable |
| Goal | inside target: Home to target direction, spacing 5, clamped into the Territory. Outside target: nearest Territory boundary point, held there | settled |
| Dead zones | goal change about 0.75, arrival about 0.5 | tunable |
| Turning | bounded about 270 degrees/s, no snap. Attack starts only within about 20 degrees of the intended direction | tunable |
| Rooting | rooted through Telegraph, impact and Recovery. Swing gap allows movement | settled |
| Collision | body `CanCollide = false`, separate fixed Hitbox, no collision group unless testing proves it needed | settled |
| Floor check | one downward check per movement step against the active environment roots (the same `Environment.folders()` the Sentry sight uses, so a removed Section is never seen), align Y, refuse a step with no floor or a step over about 2.5 studs | tunable |
| Fault | more than about 1 stud outside Territory, no floor, or below a safety Y: despawn, no reward, no credit, `WardenFault` | settled |
| No progress | in Chase or Return, not rooted, goal farther than arrival tolerance and not reduced by about 1 stud over about 3 s | tunable |
| No-progress result | Chase: abandon target and Return. Return: snap Home as recovery, log, no reward | settled |

Open combat circle: no collidable authored prop inside the usable movement circle plus a small clearance. Decals, non-colliding rubble and floor variation inside the step tolerance are allowed. If a blocker is found, move or remove it or adjust the Territory. Never add pathfinding.

#### Reward

| Rule | Value |
|---|---|
| Source | a separate Warden bonus deal. The 24-drop Cache bag is untouched |
| Per kill | exactly one ordinary Ability Pickup, **Rank 1** |
| Identity | uniform random over the four Abilities, **distinct across the Match's Wardens**, pre-dealt at setup from `wardenRandom` |
| Personalisation | none. No killer, collector, Shard or Kit logic |
| Collection | normal Pickup rules, no ownership delay, no reservation |
| Collapse, fault, Match end | no reward, no credit |
| Not in this ticket | XP, Kill Recovery, currency, Relic, second Pickup, Rank 2 or 3 world Pickups |

The reward pickup is spawned through `Pickups.drop` at the death position with the normal spread. A `source = "Warden"` tag and Warden id are carried into the normal collect, consume and clear log lines.

Wardens are also excluded from the fast-opening guaranteed-drop rule in `ranks-and-pickups.md`: a Warden reward is pre-dealt and never a Monster Family Drop Table roll.

**Loot-safe Territory.** The Warden can die anywhere in its circle and the drop spreads. The circle edge must keep a lip margin of `lethalLootMargin + maximumPickupDropSpread`, about 15 studs, **verified from the actual `Pickups.drop` spread before it is locked**. Safe reward spawn outranks the Territory size.

#### Runtime architecture and interfaces

- **FSM:** phases `Idle`, `Engaged`, `Returning`, `Dead`. A nullable `action` (`kind`, `impactAt`, `endsAt`, `resolved`, locked facing or centre, optional target id for telemetry). Telegraph, impact, Recovery, the no-target hold, the attack gap, regeneration and cooldowns are timestamps on the record.
- **Events** returned by `WardenRules` (`Acquired`, `TargetSwitched`, `AttackStarted`, `Impact`, `ActionEnded`, `Disengaged`, `ReturnStarted`, `ReturnedHome`, `RegenStarted`, `ResetComplete`, `Faulted`). The runtime does the world effects. `WardenRules` never touches `Workspace`.
- **Loop:** one Heartbeat connection for all Wardens, connected at `setup` and disconnected at `clear`. AI decisions at about 5 Hz from an accumulator. Motion, timestamps, impact, recovery, regeneration and death-fade removal every frame. **No `task.delay` and no per-Warden coroutine.** Impact is checked each frame so a 0.2 s Evade stays meaningful.
- **`Wardens` API:** `start(deps)`, `setup(matchId, occupiedPads)`, `clear()`, `step(dt)` (manual mode for specs), `removeSection(sectionId)`, `get(id)`. Dependencies are injected as for `Sentries`: `characters`, `casts`, `statuses`, `pickups`, `collapse` (its `sync`), `clock`, `markers`, `canSee`, `floorAt`, `folder`, `log`, `wardenRandom`, and the `goAt` source.
- **Records** cache `id`, `matchId`, `gateId`, `sectionId`, `home`, `territoryRadius`, `rewardAbility` from the marker at setup and never rediscover geometry.
- **`setup`** begins with `clear` and is idempotent. Order: read markers, compute Gate occupancy, select, deal, build, register, log `WardenDeal`.
- **`Match`** (stays the lifecycle owner): calls `wardens.setup(matchId, occupiedPads)` after `assignSpawns` and before placement, and `wardens.clear()` at Results. It passes only occupied pads. It never reads angles, radii or Territory. The `MatchSpec` order expectation covers this. Wardens are rebuilt after Ticket 22's restore, like Caches and Sentries.
- **Registration:** `characters.registerTarget` with `kind = "warden"`, `immovable = true`, `grantsKillRecovery = false`, `onKilled(hit)`, and a new minimal **`gate(hit) -> allowed, reason`** consulted before the default damage path. Caches keep their own `onHit`. Nothing is rewritten around the gate.
- **Gate:** evaluated at authoritative acceptance with the attacker's position then. It calls `collapse.sync(now)` first. It rejects an invalid attacker, a removed Section and an out-of-envelope attacker. It is not a policy engine.
- **Result reasons:** the combat result distinguishes ordinary hit, Shield, Evade, **Warded** and **Resisted** (damage landed, forced movement ignored). Never overload `Evaded` or a single `immune` flag.
- **Attacks:** `casts.castFromServer` with the Warden id as caster, Swing as a sector, Slam as an instantaneous circle at the locked centre. Verify and extend minimally: a players-only filter, and an arbitrary-origin 360-degree area. No separate hit-detection module.
- **`Protocol`:** `Protocol.wardenId`, an id kind the cast, attribution and `CacheRules.mayDamage` paths recognise, the tag, the folder name and the attribute names. Scoped to what the protocol needs. No generic monster namespace.
- **`finish(record, cause)`:** idempotent, first call wins. Causes `Killed`, `Collapsed`, `Fault`, `MatchEnd`. Order: mark terminal, clear the action, stop motion and AI, unregister the target, clear Statuses, do the cause-specific work, update presentation. **Only `Killed` spawns a reward**, exactly once.
- **Death presentation:** a killed Warden is already unregistered, non-damaging and non-blocking. Removal is a `removeAt` timestamp of about 0.5 s. A Collapse removal destroys it at once.
- **Collapse ordering:** `removeSection(sectionId)` is idempotent and **non-yielding**, so it fits Ticket 22's atomic removal. A lethal hit accepted before the removal boundary kills normally and its Pickup may then be swept as loose loot on doomed ground. A hit accepted at or after the boundary finds the Warden gone, with no kill and no reward. A pending impact resolves only if accepted before removal. The Warden loop and the gate both call `sync(now)`, as the Match gates and Sentry loop do.
- **Stale guards:** record identity and `matchId` checks before any side effect, with a counter and not a log line.
- **Dormancy:** with the Reference Kit no Warden exists and no loop runs.

#### Presentation

Server owns gameplay state, an anchored root, the fixed Hitbox and the attributes (`WardenState`, `WardenAction`, `WardenImpactAt`, `WardenEndsAt`, a locked centre, `WardenReward`, `WardenHome`, `WardenRadius`). Health comes from the target's replicated Health, or a normalised value if the final representation does not replicate one. Do not keep a hidden Humanoid solely for the bar. A sibling client module **`WardenVisuals`** draws everything else, from attributes only. It owns no authority and cleans up completely.

| Element | Requirement |
|---|---|
| Body | a client cosmetic rig on the authoritative root: dormant hunch, wake rise, wind-up poses, recovery lowered, death fade about 0.5 s. No smoothing until Play shows steppiness |
| Reward core | the Pickup colour and shape from the shared Ability visual data, embedded in the chest, never a collectable object before death |
| Territory ring | about 12 low-opacity arc or rune segments near the ground, subtle at Idle, brighter when Engaged, visible within about 45 studs. Marks the Territory only |
| Health bar | slim, fill only, no number or text, neutral colour, shown when Engaged, Returning or regenerating within about 40 studs |
| Wake | authoritative state is Engaged at once. The 0.4 s rise is presentation only. No hidden delay |
| Swing | locked wedge (reach 7, about 100 degrees) filling to `impactAt`, body wind-up, sharper short cue |
| Slam | locked disc (radius 5) filling to `impactAt`, heavier pose, longer deeper cue |
| Recovery | seams dim, body stays lowered |
| Evade and Shield | existing feedback unchanged |
| Warded hit | translucent shimmer, a hollow tink, a hollow marker with no damage number. An optional rate-limited "out of range" hint |
| Resisted movement | stone-set dust puff, thud, brief brace |
| Return and regen | calmer seams, bar visible, no text, a gentle pulse while healing, fill follows real Health |
| Death | seams out, core becomes the real Pickup at once, no corpse |
| Collapse or fault | no kill-like effect |
| Spectators | see the same Warden information |

Hostile Telegraphs use red, which no Ability uses. Confirm against Ticket 22's world telegraph so the two never read alike (differentiate by shape, pattern and rhythm if they do). Swing and Slam must differ without colour. Reward identity must not rely on colour alone. If two Pickup shapes are too similar, add the smallest silhouette difference. No Ability icon database. Audio uses existing suitable sounds or placeholders, and gameplay never depends on new sound assets.

#### Telemetry

Console `[Riftfall]` `key=value`, no coordinates, timed from GO. Telemetry is attempted once, failures are contained, and it never blocks gameplay.

- **Events:** `WardenDeal` (players, count, occupancy, Gates, tie candidates and result, rewards), capped `WardenAggro` and `WardenReset` (first 3 per Warden, then counted), `WardenKilled` (`by`, `ability`, `seconds`, `engagedSeconds`, `firstEngager`, `duringWarning`, `reward`, `pickup`), `WardenRemoved` (cause), `WardenFault`.
- **`WardenSummary`:** one line at Match end with per-Warden blocks. Identity (Gate, occupancy, reward). Engagement (first aggro and its cause, counts, resets by reason, unique engagers, engaged seconds, `visitors`, `closestApproach`). Damage (to the Warden, damagers and top damager, to players, players hit, player deaths). Swing and Slam as started, hits, misses and Evades, plus Shield absorbed. Ward and band (`wardedHits`, `wardedPlayers`, `wardedMaxOnePlayer`, `nearHits`, `bandHits`, `bandDamage`). Contest (`contestedSeconds`, `maxInEnvelope`, `newcomers`, `multiHitAttacks`). Route (`passByAggro`, `routeTaxDamage`). Health (`maxOutsideDistance`, `snapHome`). Outcome.
- **`passByAggro`:** a Proximity engagement with zero accepted damage to the Warden that ended in a NoTarget reset. `routeTaxDamage` is the Warden's damage to players in those engagements.
- **Reward fate:** joined offline from normal Pickup and `Equip` lines through the `source` tag and the pickup id.
- **Not logged:** per-attack events, Heartbeat or movement samples, Threat internals.

Three tiers. **Correctness (hard):** `WardenFault` = 0, snap-home = 0 in normal play, no leaked targets, connections or models, exactly one reward per legitimate kill and none for Collapse, Fault or Match end, no aggro before `goAt`, no non-player attack target, distinct rewards, a stale-rejection counter with one diagnostic if unexpectedly non-zero. **Balance evidence:** engage and kill rates by player count, Gate, occupancy and reward; fight length; damage taken; Warden-caused deaths; Swing and Slam hit, Evade and miss rates; warded rate; pass-by aggro; band use; steal rate; Collapse survival. **Role-level review triggers** are initial signals only, not read until there are enough encounters, and are listed in [`rift-warden-design.md`](rift-warden-design.md).

#### Validators (static, Edit-mode safe)

- Slot count, unique `GateId`, `SectionId` on Layer 2, `TerritoryRadius` present and positive. Every pad has a `GateId`.
- Territory circle wholly on live Layer 2 floor, clear of lethal lips by the loot-safe margin, connectors, aprons, the Crown approach and Sentries.
- No collidable authored prop inside the circle plus clearance.
- `distance(Home, every pad) >= AggroRadius + spawnSafetyBuffer` (buffer about 20, from Ticket 21), for all six pads.
- No Home Cache inside aggro. The Forward Cache outside aggro (initial extra clearance about 2, not a universal formula). Ordinary connector and apron traversal does not automatically trigger the Warden.
- `AggroRadius <= TerritoryRadius + 4`.
- An ignoring route exists from each pad to its Safe spoke and link without Dash, fighting or taking unavoidable damage (a Studio playtest requirement, with a simple sampled clearance check allowed to support it, and no pathfinding built to validate it).
- The slot offset: keep +15 to +20 degrees only if connector, apron, Territory, Forward Cache, lip, line-of-sight and ignoring-route checks pass, otherwise keep +10 degrees.
- If no valid Territory and loot margin fit the authored Layer 2 platform, flag a genuine Ticket 21 contradiction and revisit that local platform only. Do not redesign Rift Crown globally.

#### Tests

**Specs (pure and fixture-based).**
- **S1** Gate selection for 1, 2, 3, 4, 5 and 6 players on the real spread-table pad sets, with tie control.
- **S2** Reward deal: distinct, deterministic from an injected stream, and the shared `random` untouched (Cache deal and spawn assignment identical with and without Wardens).
- **S3** FSM transitions: acquire (needs line of sight), damage wake, sticky target, Threat switch with margin, absolute minimum and cooldown, the 4 s disengage, vulnerable Return, interrupted regeneration, the 3 s wait then regeneration, no instant cleanse.
- **S4** Action timeline: Telegraph, exactly one impact, rooted recovery, the Swing gap, the Slam cooldown, no retarget mid-Telegraph.
- **S5** Locomotion maths: clamp, goal, spacing as an approach target only, dead zones, the turn limit and the 20 degree gate, no-progress (Chase abandons, Return snaps).
- **S6** The stage gate at its boundary: veto means no damage, Threat or wake.
- **S7** `finish`: first wins, one reward only for `Killed`, none otherwise, idempotent, both orderings against Collapse.
- **S8** Lifecycle: `setup` and `clear` idempotence, no duplicate connections or targets, stale guards, `removeSection` idempotent and non-yielding, inert before `goAt`.
- **S9** `MatchSpec`: setup after `assignSpawns` and receives occupied pads, clear at Results.
- **S10** Validators.
- **S11** Telemetry through a fake sink: `WardenDeal`, capped events, `WardenSummary` fields, failures contained.

**Studio S1 (solo Play).**
- **P1** Idle through Countdown and the Arrival Hold: no aggro, movement, attack or damage.
- **P2** Idle, aggro, fight, disengage, Return, regeneration, full reset, with no teleport.
- **P3** Swing and Slam shapes match the Telegraph, one evaluation, players only.
- **P4** Evade negates Swing and Slam.
- **P5** Stone Guard absorbs.
- **P6** Wet then Lightning Reaction on the Warden, with Wet visible.
- **P7** Hook and Tidal damage and Wet, no movement, stone-set feedback distinct from the ward.
- **P8** Outside-fire ward: rejected, no damage, Threat or wake, ward feedback.
- **P9** Reward core readable before the fight. A kill gives exactly one Rank 1 Pickup of that Ability, collectable at once.
- **P10** No body-blocking and no push, no Movement Validation correction. Over a full fight `maxOutsideDistance` stays in tolerance and it never reaches a connector or lip.
- **P11** Match reset leaves no Warden, target or effect, with a fresh deal.
- **P12** Real log lines match what happened.

**Studio S2 (two clients).** **T1** a real 2-player Match with one Warden at the empty Gate. **T2** kill by one client, collection by the other, no reservation or delay. **T3** a Swing or Slam hits both players. **T4** a same-tick Pickup contest resolves normally. **T5** a spectator sees the same Warden state.

**Synthetic multi-Warden.** **M1** three Wardens at once with distinct rewards and no interference. **M2** the 2-Warden selection. These use a TEMPORARY hook that calls the **real** `wardens.setup` selection and deal with injected pad sets, never hand-built Wardens. Real 4 and 6 player Matches are evidence only, since the tooling cannot reliably drive them.

**Cleanup and fault.** **C1** `removeSection` and a forced Fault leave no reward, credit, leaked target, connection or model. **C2** a Warden alive at Match end is cleared with no reward.

**Method.** Exact timing lives in specs with a fake clock. Studio is for engine and visual behaviour. Run specs through `RunAll` on the Server in Play. Read [`docs/testing/studio-mcp-quirks.md`](testing/studio-mcp-quirks.md) first. Standing in Warden range between tool calls is lethal as with Sentries: arm everything, then approach, act and leave within one call.

**Evidence only (recorded, never gating).** The Swing backstep, the slam-only band share, circling at chase speed 12, a competent solo fight time and Health cost, how often the ward fires and whether it feels unfair, Safe-route pass-by, visual smoothness, the real Territory radius and loot margin, and the `WardenSummary` triggers.

#### Verify before building

- Where the target-level veto hooks into `Combat` and how a Warded or Resisted reason surfaces in `CombatResult`.
- `Casts`: players-only filtering from a server origin, and an arbitrary-origin 360-degree area.
- How `Characters.registerTarget` uses `immovable` for a model that moves.
- Whether the Warden hit pipeline reports per-target hit, Evade and Shield outcomes. If not, count through the shared `CombatResult` path.
- The elimination cause used for Sentry kills, to reuse for a Warden kill, and how Sentry hits on players are kept out of PvP kill and damage numbers (`SessionRules`), to reuse for the Warden.
- Whether Pickup and `Equip` log lines carry ids and accept a `source` tag.
- `StatusVisuals` on a non-player model.
- Whether the four Pickup shapes are distinct, and the Natural Recovery rate against the Warden damage figures.
- The actual player numbers used for tuning (Health, walk speed, Dash, Evade, Basic, Tidal, Arc Burst, Hook, Stone Guard), read from code, not these docs.
- Whether a stable Match seed exists.
- The slot-to-pad, Home Cache and Forward Cache distances at the real marker positions.

If a documented number differs slightly in code, update the tuning assumption and keep the settled combat role unless the difference materially breaks it.

#### Completion criteria

**Implementation complete** (nothing else holds the ticket open):
- All blocking specs green in `RunAll` on the Server in Play.
- The blocking Studio matrix (P1 to P12, T1 to T5, M1, M2, C1, C2) green, with the presentation checks observed.
- `WardenFault` = 0, snap-home = 0 in normal play, no leaked targets, connections or models, and exactly one reward per legitimate kill.
- Docs and results recorded in this ticket, the quirks doc updated, and every TEMPORARY hook removed (grep the word).
- A clean commit.

**Publish checkpoint** (separate manual step): the developer publishes and the version is recorded in [Checkpoints](#checkpoints).

**Blocking dependency on Ticket 22:** until both implementations coexist, only `removeSection` is blocking. The real Collapse plus Warden Studio check then becomes blocking.

#### Integration contract for Ticket 25 (added 2026-11-10)

[Ticket 25](#25-match-milestones) (Milestones, **Stage 2**) reads from this build. These are requirements, with no change to the Warden's mechanics, reward or Last Hit credit:

- A **per-player effective-damage tally per Engagement** (Health actually lost by the Warden, accepted hits only), so the Explore feat can fire when a player reaches about 25% of its maximum Health. The tally clears **when regeneration starts, on a full reset, or on despawn**, and **not** at the start of the walk home (a third party can still finish a weakened Warden during it). This is a Milestone attribution, not a reward split. Last Hit still decides the Pickup.
- Every hit a Warden deals carries a **`sourceClass`** of Warden. Warden damage is the one PvE source that counts toward Endure.
- The combat-idle signal is stamped when a Warden deals or takes damage (the stamp point is added with the shared signal).
- The Warden's per-Engagement tally is **logged** in the existing telemetry so Stage 2's evidence can use it.

#### Future work

**Regular monsters are intentionally deferred to the next PvE ticket, now specified as [Ticket 24](#24-regular-monsters).** Wardens are elite-tier, mini-boss-style guardians, not the baseline monster population. Ticket 24 starts by extracting the Warden primitives that prove genuinely reusable, and is implemented only after this ticket is built.

Also deferred: an elite Dais Warden or boss, Match XP (now Milestones, [Ticket 25](#25-match-milestones)) and Relics, patrol, respawn, a ranged or leap attack if telemetry shows kiting dominates, final art, audio and animation.

### 24. Regular Monsters

**Status: specified 2026-11-10 and reviewed adversarially the same day, not built. Implementation starts only after Ticket 23 is built** (and with it Tickets 21 and 22), because Ticket 24 reuses primitives the Warden proves and must not be built against speculative Warden infrastructure. Every design point was grilled with the developer and settled. The reasoning, rejected options and edge cases are in [`regular-monster-design.md`](regular-monster-design.md) and [ADR-0008](adr/0008-pve-actors-are-kinematic-server-models.md). This entry is the build contract. Values carry the same four labels as Tickets 22 and 23: **settled initial value**, **tunable**, **measurement-dependent**, **test-only**.

**Targets and balance expectations are not pass criteria.** The 4 to 4.5 s Basic-only kill, the 0 to 8 Health cost for a competent player, the backstep watch item, pair burst, outside-fire share, reward feel and visual smoothness produce tuning notes. Only the **Completion criteria** decide completion.

#### Summary

The **regular monster** is the Arena's baseline PvE population: weak, quick to kill and numerous, scattered near loot and routes. Its role is **ambient PvP pressure first**: it comes to players, makes noise and gets pulled into fights. A small, hidden chance of one ordinary Rank 1 Ability Pickup is a second-order reward. The first is the **Riftling**, a small melee chaser (placeholder name). It is not a weaker Warden, not a Monster Family member, and it never gates a route, Cache, Kit step or the Crown.

#### Scope

- **Step 0, shared primitives extracted from the built Warden** (see below).
- `MonsterRules` (pure): selection and deal, FSM, edge hold, reset, wake stagger, targeting glue and locomotion goals over the extracted primitives.
- `Monsters` (server): runtime, registration, motion, attacks, reward, `finish`, `removeSection`, telemetry.
- The authored pool of 12 Home markers, the static validators, and the Layer and fallback review.
- `Match` integration, `MonsterVisuals`, `Feedback` additions, `Protocol` and `Tuning` entries, the PvE actor-id predicate, specs, the Studio matrix, `CONTEXT.md`, `monsters.md` and `ranks-and-pickups.md` terms.

#### Non-goals

- A `MonsterService`, behaviour trees, `PathfindingService`, a generic enemy framework, dynamic avoidance or repulsion, pack AI, and any tuning of the Warden.
- Match XP, levels, currency, Relics, Equipment, a second reward type, Kill Recovery, permanent progression.
- Respawn, patrol, a leap, a lunge, a ranged or second attack, any Status, Root, Slow, Stun or player Knockback, cross-Shard or connector pursuit, Crown monsters.
- Monster Families, themed or weighted Drop Tables, an Elite, an outside-fire gate, a per-target Telegraph cap or a Match drop budget (each is added only on evidence).
- Final art, audio and animation, a nameplate, an overhead Health bar, a Territory ring. **No upload without explicit approval.**
- Analytics backend, heatmaps, positional sampling.
- Changing the 24-drop Cache economy, `LootRules.validate`, the Warden deal, kill attribution, or the shared Match `random` draw order.

#### Dependencies

**Blocking dependency on Ticket 23.** Ticket 24 implementation begins only when Ticket 23 is implemented, green and committed. The design and this entry may be finished earlier.

**Relies on (Ticket 21):** the six Shards and `ShardId`, pads, Home and Forward Caches, Crown Sentries, Warden slot markers, `SectionId` on markers, `goAt`, the structural validator.

**Ticket 24 adds to the built map:** 12 Home markers (two per Shard, variant `A` and `B`), each with `HomeId`, `ShardId`, `Variant`, `SectionId`, `TerritoryRadius` and two slot offsets. Exact folder and attribute layout is recorded when built, and no generic marker framework is introduced.

**Relies on (Ticket 22):** `Collapse.sync(now)`, Section removal as one non-yielding step, restore before marker discovery, and the rule that a combat event accepted before the removal boundary resolves under the old world and one accepted at or after it under the new world.

**Requires from Ticket 22:** its object-cleanup step calls `Monsters.removeSection(sectionId)`, as it does for Caches, Sentries and Wardens, and never destroys monster instances itself. This is a one-line addition to the built cleanup step.

**Relies on (Ticket 23):** `warden` target registration and `immovable`, the target-level `gate(hit)` veto, the Resisted result reason, the players-only server cast filter, the shared ground-circle cast form, PvE elimination attribution and the exclusion of PvE damage from PvP numbers, the Telegraph visual conventions, the `source` tag on Pickup log lines, and `StatusVisuals` on a non-player model.

#### Step 0: shared primitives (first implementation step)

Extract **only** what both real implementations genuinely need, from the built Warden, in **its own commit**, with every existing Warden spec and Studio check unchanged and green. Names are chosen at build and none contains "Monster":

| Primitive | Content | Class |
|---|---|---|
| Motion and Territory | pure step toward a goal with a bounded turn rate, a clamp into the Territory circle, the nearest edge point, the Home-return step, and the floor check using an injected `floorAt` (align Y, refuse a step with no floor or over the step limit, Fault conditions) | extract |
| Threat | the recent-damage window, the sticky target, the challenger margin, the absolute minimum and the switch cooldown, as pure functions | extract |
| Action record | the nullable `action` (`kind`, `impactAt`, `endsAt`, `resolved`, locked facing or centre, optional target id) and its step that yields `Impact` and `ActionEnded`. **Not** the FSM and not attack choice | extract |
| Telegraph builder (client) | the wedge and disc that fill to `impactAt` from attributes | extract |
| PvE actor-id predicate | one `Protocol` helper so `CacheRules.mayDamage`, attribution and `SessionRules` stop growing `warden` or `monster` or `sentry` cases. Not a namespace | extract |

**Reuse directly:** `characters.registerTarget`, `immovable`, the Sentry sight helper (`canSee`), `casts.castFromServer`, Status and Reaction participation, the telemetry sink. **Keep separate:** each system's `setup`, `clear`, Heartbeat loop and `removeSection`, each rules module's FSM and each `*Summary`.

**Rule.** If a supposed primitive is tangled with Warden-specific behaviour, do not force the extraction. Leave it in the Warden and implement the monster version separately, and record which primitive was not shared and why. The Warden's behaviour and numbers do not change. Review the extraction diff before any monster code is written.

#### Population and selection

Chosen once at Match setup, after `assignSpawns` and after Wardens (monsters do not depend on occupied pads). **Tunable starting rule, not scaled by player count:**

| Item | Value | Label |
|---|---|---|
| Authored pool | 12 Homes, two per Shard: variant A and variant B | settled |
| Active per Match | one Home per Shard, so 6 Homes | settled |
| Group sizes | 4 singles and 2 pairs, so 8 monsters | settled initial value, tunable |
| Crown | 0 monsters | settled |
| Maximum per Home | 2 | settled |

`monsterRandom` is a dedicated stream, injected (`deps.monsterRandom`, default `Random.new()`). It never draws from `Match`'s shared `random`, the Cache deal, the spawn assignment or `wardenRandom`. **Fixed draw order**, so specs are deterministic: (1) one A or B draw per Shard in ascending `ShardId`; (2) the two pair Shards, drawn without replacement (no Gate constraint; the assignment is evidence); (3) for each monster in Home then slot order, **always** one flag draw and one Ability draw, whether or not the flag is set. The population is never changed after GO. One life per monster is final.

#### Territory, aggro and targeting

| Rule | Value | Label |
|---|---|---|
| Home Territory | circle around the Home marker, never left | settled |
| `TerritoryRadius` | geometry-derived, about 7 (6 to 9) | measurement-dependent |
| `AggroRadius` | `Territory + 4` (never more) | tunable, geometry-validated |
| Acquire | Running and past `goAt`, player Alive and valid, inside `AggroRadius`, clear authoritative line of sight (the Sentry sight helper) | settled |
| Retention | validity and distance, not line of sight | settled |
| Damage wake | any accepted player damage, from any distance. The damager is the initial target | settled |
| `AttackReach` | 5.5 | settled initial value |
| Contestable | `distance(Home, target) <= TerritoryRadius + AttackReach` | settled rule |
| Initial target | nearest valid player, tie by id order | settled |
| Threat, switch | the extracted Threat primitive with the Ticket 23 values | tunable |
| Wake stagger | one random 0.3 to 0.7 s before the **first** attack after each acquisition or re-engagement (Idle to Engaged, or Returning to Engaged). Not after a target switch, and never ongoing jitter | settled rule, tunable |
| Edge hold | 1.5 s, starting only once the monster has arrived at the nearest Territory edge point and the target is still beyond `Territory + AttackReach` | settled |
| No-target hold | 1.5 s in place, then Return, when the target dies, leaves or is Eliminated and no other valid target exists | tunable, an assembly default |
| Return | normal speed, targetable and damageable. **Accepted hits do not re-engage it, from any attacker.** Only a valid player inside the Territory re-engages | settled |
| Reset on reaching Home | Health full at once, Statuses, Threat and target cleared, Idle. No gradual regeneration | settled |

The monster is inert before `goAt`: no aggro, movement, attack or targeting. It must check `goAt` as Sentry targeting does, not rely on the Match phase. A committed attack never retargets.

**Between the circle and `Territory + AttackReach`** the target is contestable: the monster fights it from the edge, and there is no hold and no reset.

**Outside fire.** There is no gate. Damage from beyond `Territory + AttackReach` is accepted normally, applies Statuses and Reactions, and wakes the monster. The reset is the cost: after the hold and the Return the monster returns to full Health, so a ranged player must burst it down in that window. Telemetry names this `outside*` and defines it as **beyond `Territory + AttackReach`** (uncontestable), because inside that range the monster can fight back. Outside fire that also lands during the Return is counted and never re-engages.

#### Combat

**Identity:** a small melee chaser with one readable claw swing. Exactly one attack.

| Item | Value | Label |
|---|---|---|
| Shape | flat frontal sector, reach 5.5, about 50 degrees | settled initial value, tunable |
| Lock | facing at Telegraph start, no tracking afterwards | settled |
| Telegraph | 0.45 s, rooted | tunable (toward 0.55 s before lowering damage) |
| Impact | one authoritative evaluation at `impactAt` | settled |
| Damage | 8 | tunable |
| Recovery | 0.8 s rooted, then a 0.4 s movement-allowed attack gap, so about 1.65 s per cycle | tunable |
| Attack start | target inside reach and within about 20 degrees of facing, else chase | tunable |
| Health | 45 (range 40 to 50) | tunable (toward 40 first if kills run long) |

A hit shape hits every valid player inside it once per attack, never monsters, Wardens, Sentries, Caches, Pickups or map objects (the explicit players-only filter). The monster chooses one target and aims at one target. There is **no** single-target damage filter and no deliberate cleave. An impact resolves only while the Match is Running and past `goAt`.

| Item | Value | Label |
|---|---|---|
| Damage to players | normal damage and Shield pipeline, no scaling by player count, Rank or time | settled |
| Evade, Stone Guard | accepted Dash negates it, Shield absorbs, both with normal feedback and no monster exception | settled |
| Statuses, Knockback, Root, Stun against players | none | settled |
| Wet, Reactions, Arc Burst | work on the monster normally | settled |
| Forced movement against it | immune (Tidal Knockback, Hook pull move nothing), damage and Statuses still apply, shared Resisted result | settled |
| Falling | cannot be displaced or fall, no monster Void kill | settled |
| Kill Recovery | none | settled |
| Kill credit | `hit.attackerId` of the accepted lethal player hit (Last Hit), attribution only | settled |

A player killed by a monster is eliminated with **no killer** and no Kill Recovery, using the Sentry and Warden convention. Monster damage to players is real combat damage, kept out of PvP kill and damage numbers (as Sentries and Wardens are in `SessionRules`) and logged separately. **Kill attribution is not changed in this ticket**, including when a monster finishes a player another player had damaged (`finishedAfterPlayerDamage` is evidence only).

**Dogpile.** No simultaneous-attacker cap. A maximum of 2 per Home, the first-attack stagger, and overlap telemetry. Add a per-target Telegraph cap only if testing shows pairs routinely sync into unfair bursts.

#### Locomotion

Anchored, kinematic, server-moved ([ADR-0008](adr/0008-pve-actors-are-kinematic-server-models.md)), over the extracted Motion primitive.

| Item | Value | Label |
|---|---|---|
| Chase speed | 13 studs/s (range 12 to 14.5), below the player's 16 so a player can always disengage | tunable |
| Approach spacing | about 3.5 studs, an approach target and never a repulsion radius | tunable |
| Goal | inside target: Home to target direction, spacing 3.5, clamped into the Territory. Beyond contest: the nearest Territory edge point, held there | settled |
| Pair offset | each pair member's goal adds a small lateral offset by slot index (about 1.5 studs, perpendicular to Home to target), so two members never share an exact attack position. Idle positions are the authored slot offsets. **No dynamic avoidance or repulsion** | tunable |
| Dead zones, turning, rooting, collision, floor check, Fault, no progress | the Warden's values and results as built (an assembly default), with the monster's own Territory | tunable |
| Fault result | despawn, no reward, no credit, `MonsterFault` | settled |
| No-progress result | Chase: abandon target and Return. Return: snap Home as recovery, log, no reward | settled |

Open combat circle: no collidable authored prop inside the usable movement circle plus a small clearance. Decals, non-colliding rubble and floor variation inside the step tolerance are allowed. If a blocker is found, move it or the Home. **Never add pathfinding.**

#### Reward

| Rule | Value |
|---|---|
| Source | a separate monster bonus deal. The 24-drop Cache bag and the Warden deal are untouched |
| Per monster | an independent hidden flag at setup, **12.5%** (`Config`) |
| Per flagged kill | exactly one ordinary Ability Pickup, **Rank 1**, identity uniform over the four Abilities, drawn at setup, **repeats allowed** |
| Cap | none. No pity, no guarantee after N kills |
| Budget | about 0.5 expected drops per Match at 4 kills of 8, and 1.0 at a full clear, below the guaranteed Warden supply |
| Personalisation | none. No killer, collector, Shard or Kit logic |
| Collection | normal Pickup rules, no ownership delay, no reservation |
| Collapse, Fault, Match end | no drop, no credit |
| Not in this ticket | XP, currency, Relic, Kill Recovery, a second Pickup, Rank 2 or 3 world Pickups |

The flag and Ability live only in the server record. **Nothing replicates before death**, and flagged and unflagged monsters look identical. The Pickup is spawned through `Pickups.drop` at the death position with the normal spread, with `source = "Monster"` and the monster id in the normal collect, consume and clear log lines.

**Loot-safe Territory.** The Territory edge keeps a lip margin of `lethalLootMargin + maximumPickupDropSpread`, about 15 studs, **verified from the actual `Pickups.drop` spread before it is locked**.

**The old fast-opening rule does not apply.** Regular monsters, like Wardens, are excluded from the rule in `ranks-and-pickups.md` that an empty Ability slot guarantees the next eligible monster kill. A regular monster's reward is a pre-dealt flag and never a Monster Family Drop Table roll.

#### Placement and validators (static, Edit-mode safe)

All distances are **starting values, greybox-validated against the built map**.

| Relative to | Rule |
|---|---|
| Spawn pads | at least `max(45, AggroRadius + 20)` from **every** pad, own Shard's included. 45 is Ticket 21's `pveSlotClearance` and is the stricter value here (aggro + 20 is about 31 to 33), so it governs |
| Home Caches | Home at least 20, so the Cache and its drop spread (about 5) sit outside aggro with margin |
| Forward Cache | Home about 20 to 32 away. A neighbour that fights can drift into, never a guard. Cache standing spots and drop spreads stay outside the aggro ring |
| Warden | the monster's aggro circle (`Territory + 4`) and the Warden's (`Territory + 4`) must not intersect, plus 2. Checked against **all three Warden slots** whether or not occupied this Match |
| Crown Sentries | at least 50 from every Sentry |
| Lethal lips | the Territory edge at least 15 from any lip (the loot-safe margin) |
| Connectors and aprons | the aggro ring clear of every connector and apron, plus 2. A bridge traversal never wakes a monster |
| Section | the whole circle on live floor of **exactly one** Section. A circle straddling Sections fails |
| Floor and props | floor steps at most about 2.5 sampled across the circle, no collidable authored prop inside the circle plus clearance, and the two slot offsets inside the circle with at least 2 spare |
| Radius | `TerritoryRadius` present, about 6 to 9, and `AggroRadius <= TerritoryRadius + 4` |
| Counts | exactly two Homes per Shard, one A and one B, and `Variant` values unique per Shard |

- **An ignoring route exists** from every pad toward the Crown that never enters an aggro ring, without Dash, fighting or unavoidable damage. This is a Studio playtest requirement, with a simple sampled clearance check allowed to support it. No pathfinding is built to validate it.
- **Section and layer.** Prefer a Layer 2 Home on the inward, Forward side where geometry allows, and allow Layer 1 where necessary. **Layer 2 is not hard-required.** A Layer 1 Home lives to the Layer 1 removal, a Layer 2 Home to the Layer 2 removal. No Layer 3 or Crown Home.
- **Feasibility fallback**, in this order: (1) relax the Forward Cache 20 to 32 relationship; (2) allow or move variant B further outward, or onto Layer 1. **Never relax spawn safety, Home Cache safety or the lethal-lip margin merely to force a slot.**
- If a Shard genuinely cannot support **one valid Home at all**, that is a **Ticket 21 map-placement contradiction**. Review that local geometry. Do not silently drop the Shard from the population rule, and do not redesign the Arena globally.

#### Runtime architecture and interfaces

- **FSM:** phases `Idle`, `Engaged`, `Returning`, `Dead`, in `MonsterRules`. Telegraph, impact, Recovery, the attack gap, the wake stagger, the edge-hold start and the no-target hold are timestamps on the record. The nullable `action` is the extracted record. `Engaged` with a target beyond contest moves to the edge point and starts the hold on arrival.
- **Events** returned by `MonsterRules` (`Acquired`, `TargetSwitched`, `AttackStarted`, `Impact`, `ActionEnded`, `HoldStarted`, `ReturnStarted`, `ReturnedHome`, `ResetComplete`, `Faulted`). The runtime does the world effects. `MonsterRules` never touches `Workspace`.
- **Loop:** one Heartbeat connection for all monsters, connected at `setup` and disconnected at `clear`. AI decisions at about **5 Hz** from an accumulator (tunable up to about 10 Hz only if acquisition feels late). Motion, timestamps, impact and removal every frame. **No `task.delay` and no per-monster coroutine.** Impact is checked each frame so a 0.2 s Evade stays meaningful. An Idle monster does only cheap proximity checks.
- **`Monsters` API:** `start(deps)`, `setup(matchId)`, `clear()`, `step(dt)` (manual mode for specs), `removeSection(sectionId)`, `get(id)`. Dependencies are injected as for `Sentries` and `Wardens`: `characters`, `casts`, `statuses`, `pickups`, `collapse` (its `sync`), `clock`, `markers`, `canSee`, `floorAt`, `folder`, `log`, `monsterRandom`, and the `goAt` source.
- **Records** cache `id`, `matchId`, `homeId`, `shardId`, `sectionId`, `home`, `slotIndex`, `territoryRadius` and the hidden reward from the marker and the deal at setup. They never rediscover geometry.
- **`setup`** begins with `clear` and is idempotent. Order: read markers, select Homes, deal, build, register, log `MonsterDeal`.
- **`Match`** (stays the lifecycle owner): calls `monsters.setup(matchId)` after `assignSpawns` and after `wardens.setup`, and before placement, and `monsters.clear()` at Results. Monsters are rebuilt after Ticket 22's restore, like Caches, Sentries and Wardens. It never reads angles, radii or Territory. The `MatchSpec` order expectation covers this.
- **Registration:** `characters.registerTarget` with `kind = "monster"` (explicit, never inferred from names or folders), `immovable = true`, `grantsKillRecovery = false`, `onKilled(hit)`, and a `gate(hit)` containing **only** the Collapse check: it calls `collapse.sync(now)` first and refuses a hit when the record is gone. It has **no range check**. If the Ticket 23 build shows the existing hit gates already sync before any target is resolved, the gate can be dropped. It is also where an outside-fire gate would hook in if evidence ever asks for one.
- **Result reasons:** Resisted (damage landed, forced movement ignored) is the shared one from Ticket 23. A monster never produces a Warded result.
- **Attacks:** `casts.castFromServer` with the monster id as caster, the claw as a sector, using the players-only filter from Ticket 23. No separate hit-detection module.
- **`Protocol`:** `Protocol.monsterId` (an id kind the cast, attribution and `CacheRules.mayDamage` paths recognise), the tag, the folder name and the attribute names (`MonsterState`, `MonsterAction`, `MonsterImpactAt`, `MonsterEndsAt`, a locked facing, `MonsterHome`, `MonsterRadius`, `MonsterDamageStage`), through the one actor-id predicate. No generic monster namespace, and **no reward attribute**.
- **`finish(record, cause)`:** idempotent, first call wins. Causes `Killed`, `Collapsed`, `Fault`, `MatchEnd`. Order: mark terminal, clear the action, stop motion and AI, unregister the target, clear Statuses, do the cause-specific work, update presentation. **Only `Killed` on a flagged record spawns a reward**, exactly once.
- **Death presentation:** a killed monster is already unregistered, non-damaging and non-blocking, and removal is a short `removeAt` timestamp. A Collapse removal destroys it at once.
- **Collapse ordering:** `removeSection(sectionId)` is idempotent and **non-yielding**, so it fits Ticket 22's atomic removal. A lethal hit accepted before the removal boundary kills normally and its Pickup may then be swept as loose loot on doomed ground. A hit accepted at or after the boundary finds the monster gone, with no kill and no drop. A pending impact resolves only if accepted before removal.
- **Stale guards:** record identity and `matchId` checks before any side effect, with a counter and not a log line.
- **Dormancy:** with the Reference Kit no monster exists and no loop runs.

#### Presentation

Server owns gameplay state, an anchored root, the fixed Hitbox and the attributes. A sibling client module **`MonsterVisuals`** draws everything else, from attributes only. It owns no authority and cleans up completely. Health comes from the target's replicated Health exactly as built for the Wardens. **There is no Health bar and no nameplate.**

| Element | Requirement |
|---|---|
| Body | a small, low-profile rift crawler about 3.5 studs long: dark stone, violet-teal emissive seams, an oversized forelimb, built from Parts, no uploaded mesh |
| Damage state | three coarse stages (healthy, wounded, nearly dead) at about 66% and 33%, shown by the seams dimming and cracking. Deliberately not an exact Health substitute |
| Territory | **no ring** by default. Hold-and-return body language shows the leash |
| Idle | crouched, dim seams, a slow breathing pulse, no wandering |
| Wake | authoritative state is Engaged at once. The rise, hiss and one seam flare (about 0.25 s) are presentation and play **during the wake stagger**, so players see "it noticed me" before the first Telegraph |
| Chase and Return | a low scuttle with brighter seams. Return: seams dim and a slower pace with a short stand-down cue. State reads from pose and motion, not colour alone |
| Claw | the locked wedge (reach 5.5, about 50 degrees) filling to `impactAt` over 0.45 s, lighter than the Warden's, with the forelimb wind-up, a short intake cue, then a slash flash and snap at impact. Built by the shared Telegraph builder |
| Hit, Evade, Shield | existing feedback unchanged, plus a 0.1 s seam flicker and flinch with no displacement and no hit-stop |
| Resisted movement | the shared Resisted response from Ticket 23, distinct from Evade and Shield |
| Death | about 0.4 s: the body collapses, seams flare then go out, a dust puff, the body fades by about 1 s. A distinct kill-confirm for the killer. No corpse, no loot sparkle |
| Drop | **flagged and unflagged monsters look identical before death.** On a flagged kill the real Pickup appears at once, plus the **Cache-break reward cue**, visual and audio |
| Collapse or fault | a quiet fade with no kill-like effect |
| Spectators | see the same monster information |

Hostile Telegraphs use red, which no Ability uses. Confirm against Ticket 22's world telegraph so the two never read alike. Visuals and sounds run only within about 60 studs of the camera. At most six cues (wake, wind-up, impact, hit, death, return), each distinct from the Sentry and Warden cues. **Status visuals:** if Ticket 23 already generalised Wet and Reaction visuals to non-player PvE targets, reuse them. If not, do not create a second competing implementation: fix or extract the shared PvE Status presentation once. Audio uses existing suitable sounds or placeholders. **No upload without explicit approval.**

**Small-screen readability is blocking** (see the completion criteria).

#### Telemetry

Console `[Riftfall]` `key=value`, no coordinates, timed from GO, attempted once, failures contained, and it never blocks gameplay. **Keep it lightweight:** if the per-Home summary becomes complicated, prioritise the counters that directly answer gameplay questions. No aggregation system for eight monsters.

- **Events (capped at about 32 per Match):** `MonsterDeal` (players, the selected Homes with variant and size, the pair Shards, and `home/id -> none|ability`, enough to reconstruct which authored monster carried which hidden reward; server console only), `MonsterKill` (id, seconds, killer token, whether the lethal hit was outside, whether any player was hit during the engagement, drop), `MonsterPlayerDeath` (`alone`, `multiMonster`, `finishedAfterPlayerDamage`), `MonsterFault`.
- **`MonsterSummary`:** one line at Match end with a header and one compact block per Home. Header: `players`, `spawned`, `homes`, `pairs`, `activeAtEnd`. Per Home: first aggro and its cause (`Proximity`, `DamageInside`, `DamageOutside`), kills and engaged seconds to kill, unique engagers, resets by reason (`Unreachable`, `NoTarget`, `Other`) and total hold seconds, Collapse removals by layer (`removedEngaged`, `removedUntouched`), `outsideFirstHits`, `outsideHits`, `outsideDamage`, `outsideKills`, attacks started, hits, misses, Evades and Shield absorbed, damage to players and `playersHit` with context (`solo`, `pvpContext`, `multiMonster`), overlap (`overlapImpacts`, `pairBurstMax`, `maxConcurrentTelegraphs`), reward (`flagged`, `flaggedKilled`, `dropped`, `flaggedUnkilled`), and health (`faults`, `escapes`, `snapHome`, `maxOutsideDistance`).
- **Derived fields:** `accidentalEngagements` (a Proximity engagement with zero accepted damage to the monster that ended in a NoTarget reset) with `routeTaxDamage`, and `flawlessKills` (a kill with no damage ever dealt to a player).
- **Reward fate:** collection is joined offline from normal Pickup and `Equip` lines through the `source` tag and the pickup id, never duplicated here.
- **Not logged:** positions, per-attack events, Heartbeat or movement samples, Threat internals.

Three tiers. **Correctness (blocking, hard):** `MonsterFault` = 0 and `snapHome` = 0 in normal play, with `maxOutsideDistance` within the Fault threshold; the population exact (6 Homes, 4 singles, 2 pairs, 8 monsters, one variant per Shard); no aggro or attack before `goAt`; no hit on a non-player (`nonPlayerHits` = 0); **`dropped` equals `flaggedKilled`**, exactly one Pickup per flagged legitimate kill and none for an unflagged kill, Collapse, Fault or Match end; `reengagedByOutsideHitDuringReturn` = 0; `holdShortResets` = 0 (a reset whose hold ran under 1.5 s from arrival at the edge); `resetNotFull` = 0; `aliveOnRemovedSection` = 0; no leaked targets, connections or models after `clear`; the shared `Match` random, Cache deal and Warden deal unchanged with and without monsters; a stale-rejection counter with one diagnostic if unexpectedly non-zero; no coordinates in any line.

**Balance evidence** (observed, never pass or fail): kill rate and engaged time to kill against the 4 to 4.5 s target, damage taken per engagement by context, attack hit, miss and Evade rates, outside-fire and reset rates, overlap and burst figures, flawless kills, accidental engagements, drop counts and wasted flags, all split by player count and Home variant.

**Role-level review triggers.** Initial signals only, not read until there are about 10 Matches with a spread of player counts, and **combined with actual playtest feedback**. Never decide balance from a tiny sample.

| Question | Initial signal | First lever |
|---|---|---|
| Ignored? | kills under about 25% of spawned, or most Homes `removedUntouched` | placement closer to routes, then count |
| Farmed? | kills over about 75% in most Matches | count or reward chance, not damage |
| Mostly route tax? | `accidentalEngagements` over about half of engagements | aggro or placement, then the ignoring route |
| Outside fire dominating? | `outsideKills` over about 30% of kills | the Warden-style `gate(hit)` for monsters |
| Unfair pair burst? | repeated `overlapImpacts`, or `pairBurstMax` at about 40 Health or more | a per-target Telegraph cap |
| Meaningful PvP pressure? | `pvpContext` under about 10% of monster damage | placement and aggro |
| Monsters stealing PvP kills? | `finishedAfterPlayerDamage` recurring | review the attribution rule, in a later ticket |
| Lethal alone? | any `MonsterPlayerDeath` with `alone` from a single monster | damage, then the Telegraph |
| Reward feel? | average drops per Match under about 0.3 or over about 1.0 | `p`, read with tester feedback (no drop in about 59% of 4-kill Matches is expected) |

#### Tests

**Specs (pure and fixture-based).**
- **S1** Deal and selection: one variant per Shard, 6 Homes, 4 singles and 2 pairs, 8 monsters, deterministic from an injected stream with the fixed draw order (both draws always taken), flag rate at the `p` boundary, Ability uniform with repeats allowed, the shared `Match` random, Cache deal, spawn assignment and `wardenRandom` untouched.
- **S2** FSM: acquire (needs line of sight), damage wake from any distance with the damager as target, sticky target and Threat switch through the primitive, the no-target hold, vulnerable Return, **outside hits during Return do not re-engage**, a valid player inside the Territory does, the full reset on arrival (Health, Statuses, Threat, target, Idle) and no gradual regeneration.
- **S3** Contest and hold: the `Territory + AttackReach` boundary, the edge-point goal, the hold starts **only on arrival at the edge point**, a target that becomes reachable first cancels it, a brief boundary crossing while the monster is still far does not reset, and a target between the circle and the contest distance is fought with no hold.
- **S4** Action timeline: Telegraph 0.45 s, exactly one impact, rooted recovery, the movement-allowed gap, about 1.65 s cycle, no retarget mid-Telegraph, the wake stagger **only on the first attack after acquisition or re-engagement**, none after a target switch, and a deterministic cycle afterwards.
- **S5** Locomotion: the extracted maths (clamp, goal, spacing as an approach target only, turn limit, the 20 degree attack gate), the pair lateral offset, no shared exact attack position, no-progress results (Chase abandons, Return snaps), and the Fault conditions.
- **S6** Registration and `gate`: `kind = "monster"`, `immovable`, no Kill Recovery, the Collapse-only gate accepts before the removal boundary and refuses at or after it with no range check, and a forced-movement hit returns Resisted with damage and Statuses applied.
- **S7** `finish`: first wins, one Pickup only for `Killed` on a flagged record, none otherwise, idempotent, both orderings against Collapse, a lethal hit inside the Warning accepted normally.
- **S8** Lifecycle: `setup` and `clear` idempotence, no duplicate connections or targets, stale guards, `removeSection` idempotent and non-yielding, inert before `goAt`, inert with the Reference Kit.
- **S9** `MatchSpec`: setup after `assignSpawns` and `wardens.setup`, before placement, rebuilt after the Collapse restore, clear at Results.
- **S10** Validators, including the fallback ordering, the one-valid-Home-per-Shard contradiction report, and a deliberately failing fixture for each rule (a straddled Section, a lip margin, a connector overlap, a Warden aggro overlap).
- **S11** Telemetry through a fake sink: `MonsterDeal` reconstructs `home/id -> none|ability`, capped events, `MonsterSummary` fields, the `outside*` definition, failures contained, no coordinates.
- **S12** Extraction regression: every existing Warden spec passes **unchanged** after Step 0, the primitives have their own specs, and a note records any primitive that was not shared.
- **S13** The PvE actor-id predicate recognises Sentry, Warden and monster ids in `CacheRules.mayDamage` and attribution, and `SessionRules` keeps monster damage and kills out of PvP numbers.

**Studio S1 (solo Play).**
- **P1** Idle through Countdown and the Arrival Hold: no aggro, movement, attack or damage.
- **P2** Idle, wake (the cue plays during the stagger), chase, claw, disengage, Return, full reset, with no teleport.
- **P3** The wedge matches the Telegraph, one evaluation, players only, and a second player inside the wedge is also hit.
- **P4** Evade negates the claw. **P5** Stone Guard absorbs.
- **P6** Wet then Lightning Reaction on a monster, with the Status visible through the single shared presentation.
- **P7** Hook and Tidal damage and Wet, no movement, and the shared Resisted feedback distinct from Evade and Shield.
- **P8** Outside fire: damage accepted from beyond `Territory + AttackReach`, the edge point reached, the 1.5 s hold timed from arrival, the Return, hits during it not re-engaging, a full reset on arrival, a burst that kills in the window accepted, and `outside*` counters correct.
- **P9** A no-drop kill has the full kill-confirm and no Pickup. A flagged kill (through a TEMPORARY hook that calls the **real** `monsters.setup` with an injected stream, never hand-built monsters) drops exactly one Rank 1 Pickup of the dealt Ability, collectable at once with the Cache-break cue. Flagged and unflagged monsters look identical before death.
- **P10** No body-blocking, no push and no Movement Validation correction. Over a full fight `maxOutsideDistance` stays in tolerance and the monster never reaches a connector or lip.
- **P11** Match reset leaves no monster, target or effect, with a fresh deal.
- **P12** Real log lines match what happened.
- **P13 (blocking)** **Small-screen readability:** on the Studio device emulator at a phone-sized viewport, or on a phone, the monster, its wake cue, the claw wedge, a pair's two overlapping wedges and the damage stages are readable. The result is recorded as a developer keep or replace judgement.
- **P14** A pair's independent cycles and the first-attack stagger observed, with the overlap counters recorded.

**Studio S2 (two clients).** **T1** a real 2-player Match with all 8 monsters. **T2** one client kills a flagged monster and the other collects the Pickup with no reservation or delay. **T3** a claw hits both players when both stand in the wedge. **T4** a spectator sees the same monster state.

**Real map placement.** **M1** run `setup` on the real built Rift Crown many times: 6 Homes each time, one per Shard, every A and B variant exercised, every validator green, no Shard without a valid Home. **M2** the ignoring route from every pad checked in Studio. These are evidence for the placement rules and the fallback order, and any failure is a map contradiction to review, never silently ignored.

**Coexistence.** **X1** Wardens and monsters in one Match: no aggro-circle overlap, no interference, distinct deals, and the Warden specs and Studio matrix still green.

**Cleanup and fault.** **C1** `removeSection` for each layer, a forced Fault and Match end leave no Pickup, credit, leaked target, connection or model. **C2** a monster alive at Match end is cleared with no drop.

**Method.** Exact timing lives in specs with a fake clock. Studio is for engine and visual behaviour. Run specs through `RunAll` on the Server in Play. Read [`docs/testing/studio-mcp-quirks.md`](testing/studio-mcp-quirks.md) first. Standing in monster range between tool calls is damaging as with Sentries: arm everything, then approach, act and leave within one call. Real 4 and 6 player Matches are evidence only, since the tooling cannot reliably drive them.

**Evidence only (recorded, never gating).** The claw backstep, the Basic-only kill time, outside-fire share, pair burst and overlap, the real Territory radius, loot margin and Layer split, reward counts against the 0.5 expectation, route tax, visual smoothness, and the `MonsterSummary` triggers.

#### Implementation notes (not part of the gameplay specification)

- Follow `CLAUDE.md`'s division. The main session builds the rules, the runtime, registration, the actor-id predicate, the Collapse hook and the Match integration, and reviews Step 0. `studio-implementer` writes mechanical specs from the written list, the Home markers from an exact brief, and cosmetic `MonsterVisuals`. `studio-tester` runs the Play matrices and returns compact tables. One agent drives Studio at a time.
- **Build order:** (0) **extract the shared primitives from the built Warden and its specs, in its own commit, Warden specs unchanged and green;** (1) `MonsterRules` with its specs; (2) the `Monsters` runtime with injected dependencies, registration, the actor-id predicate, the Collapse hook, and the removal, finish and reset specs; (3) the Home markers and the validator extension on the real map; (4) the Match integration; (5) `MonsterVisuals` and the `Feedback` additions, reusing the shared Telegraph builder and Status presentation; (6) telemetry; (7) the Studio matrix, M1 and M2, X1, and P13; (8) cleanup, documentation and the commit. Each step is testable before the next.
- Do not implement against speculative Warden infrastructure. If Ticket 23's built interfaces differ from this entry's assumptions, adjust the monster side and record the difference.

#### Verify before building

- Which Ticket 23 primitives were actually built as separable, pure code, and which are tangled with Warden-only behaviour (Step 0's outcome).
- Whether the existing hit gates already call `collapse.sync(now)` before a target is resolved, which decides whether the monster `gate` is needed.
- How the Warden represents Health for the shared Resisted and damage-stage presentation, and whether `StatusVisuals` already works on a non-player model.
- The actual player numbers used for tuning (Health, walk speed, Dash, Evade, Basic, Tidal, Arc Burst, Hook, Stone Guard, **and the Rank 2 values**), read from code, not these docs. If Rank 2 adds about 10%, the Tidal then Arc combo can kill outright, which is accepted but must be confirmed.
- The actual `Pickups.drop` spread, to verify the loot-safe margin.
- Which existing record gives the last player damage within a window, for `finishedAfterPlayerDamage` and `pvpContext`, and whether the hit pipeline reports per-target hit, Evade and Shield outcomes (if not, count through the shared `CombatResult` path).
- Whether Pickup and `Equip` log lines carry ids and accept a `source` tag.
- The Home Cache, Forward Cache, Warden slot, Sentry and pad distances at the real marker positions, and whether a valid Home exists on every Shard.
- Whether Ticket 23's built `Match` order calls `wardens.setup` where this entry assumes.

If a documented number differs slightly in code, update the tuning assumption and keep the settled role unless the difference materially breaks it.

#### Completion criteria

**Implementation complete** (nothing else holds the ticket open):
- Step 0 landed as its own commit, with every Warden spec unchanged and green.
- All blocking specs (S1 to S13) green in `RunAll` on the Server in Play.
- The blocking Studio matrix (P1 to P13, T1 to T4, M1, M2, X1, C1, C2) green, with the presentation checks observed. **P13 small-screen readability is blocking**, recorded as a developer judgement.
- The correctness tier above holds: `MonsterFault` = 0, `snapHome` = 0, no leaked targets, connections or models, exactly one Pickup per flagged legitimate kill, and the `dropped` equals `flaggedKilled` check.
- Docs and results recorded in this ticket, the quirks doc updated, and every TEMPORARY hook removed (grep the word).
- A clean commit.

**Publish checkpoint** (separate manual step): the developer publishes and the version is recorded in [Checkpoints](#checkpoints).

#### Integration contract for Ticket 25 (added 2026-11-10)

[Ticket 25](#25-match-milestones) (Milestones, **Stage 2**) reads from this ticket. These are requirements on this build, with no change to its mechanics:

- A killed Riftling is reported with its **killer (the Last Hit)** and its **Home's Shard**, so the Explore feat "Riftling on another Shard" can be judged. A Collapse despawn, a Fault or a Return gives no credit (already the case).
- Every hit a Riftling deals carries a **`sourceClass`** of Riftling, so Endure can exclude it. Riftling damage does not count toward Close Call or Pressured Exit.
- The combat-idle signal is stamped when a Riftling deals or takes damage (the stamp point is added with the shared signal).

#### Future work

Deferred, each on evidence from the run test: a ranged or mixed regular-monster family, Monster Families with themed Drop Tables, Relics (Match progression is Milestones, [Ticket 25](#25-match-milestones), not XP), an outside-fire gate for monsters, a per-target Telegraph cap, a Match drop budget, an attribution rule for monsters finishing damaged players, Crown monsters, patrol and respawn if telemetry shows the Arena empties, an elite Dais Warden or boss, final art, audio and animation.

#### Assembly defaults (not individually grilled, recorded so they are reviewable)

The no-target hold of 1.5 s; the pair lateral offset of about 1.5 studs; the turn rate, dead zones, floor step limit, Fault and no-progress values copied from the Warden; the 5 Hz decision cadence for 8 actors; the telemetry event cap of about 32; the 60-stud effect distance; and the definition of telemetry `outside*` as beyond `Territory + AttackReach`.

#### Review notes (2026-11-10)

An adversarial pass against Tickets 21 to 23, ADR-0008 and the existing combat, Pickup and Collapse rules found **no blocking contradiction**. Fixes applied to the documents:

- **Spawn clearance:** Ticket 21's `pveSlotClearance` (45) and its superseding `aggro + spawnSafetyBuffer` rule both exist. Monster Homes use the stricter, `max(45, aggro + 20)`, so 45 governs, and the wording no longer implies the superseded constant alone.
- **The fast-opening guaranteed-drop rule** in `ranks-and-pickups.md` contradicted Point 1's no-guarantee rule. Regular monsters are now explicitly excluded, as Wardens already were. Ticket 18 already had no guaranteed-first-kill rule.
- **`monsters.md`** called the Stone/Rift Knockback slam a trait of "future regular monsters". The Riftling has no Knockback, so that is now stated as a trait of a later family, and the Riftling is recorded as a family-less first monster.
- **`monsters.md` and ADR-0008** described direct movement with pathfinding when stuck. ADR-0008 already rejects that for PvE actors. `monsters.md` now says the Riftling follows ADR-0008, and the ADR notes its second user.
- **Collapse safety without a range gate:** a monster has no outside-fire gate, so the ticket gives it a Collapse-only `gate(hit)` (or drops it if the existing hit gates already sync), so a hit at or after the removal boundary cannot land on a removed monster.
- **"Outside" in telemetry** is defined as beyond `Territory + AttackReach`, because inside that range the monster can fight back.
- **Return rule difference from the Warden** (hits during Return do not re-engage a monster) is stated explicitly as intended, and the design doc records why.
- **Glossary:** `CONTEXT.md` gains **Riftling**, and the Regular monster entry now describes the Home and the approved role.

Accepted, with evidence to follow: pair burst, outside-fire share, Layer 2 fit, monsters feeling pointless (about 59% of 4-kill Matches pay nothing by design), and a monster finishing a damaged player.

Correctly deferred: see Future work. Review result: **no new ADR required.** ADR-0008 already covers the kinematic model, and this ticket's decisions are tunable gameplay and Rules plus system work.

### 25. Match Milestones

**Status: specified 2026-11-10 and reviewed adversarially the same day, not built, and conditional on gameplay evidence.** The ticket is **staged**: Stage 1 may be built now, Stage 2 only after Tickets 21 to 24 are built, and Stage 3 (Reserve) only if run tests justify it. Every design point was grilled with the developer and settled. The reasoning, rejected options and edge cases are in [`milestone-design.md`](milestone-design.md) and [ADR-0009](adr/0009-match-progression-through-one-time-milestones.md). This entry is the build contract. Numbers are **tunable** unless marked fixed, and are starting values, not balance guarantees. Exact module names and folder layout are provisional and recorded when built.

**Targets and balance expectations are not pass criteria.** Riposte's value, Catch Breath's value, Lane completion rates, the 2-player and full-lobby behaviour, and stacking produce tuning notes. Only the **Completion criteria** decide completion.

#### Summary

During a Match a player can earn up to three **Milestones**, one per **Lane**: **Explore**, **Contest** and **Endure**. Each is completed once by a feat that comes out of ordinary play, and grants one defined tactical reward: **Reserve**, **Riposte** and **Catch Breath**. A reward becomes usable once 60 s have passed since GO and the player has had an out-of-combat moment since earning it. Everything is Match-local. There is no XP, level, currency, reward menu or permanent progression. A Milestone is not an Ability Rank, and no reward touches Rank.

**On agency:** with one defined reward per Lane, a player is not choosing a progression path. They are earning three predefined tools through different kinds of play. This is worth building only if the rewards create enough depth, which Stage 1 exists to test.

#### Stages and gates

| Stage | Contents | Can be built | Evidence to proceed |
|---|---|---|---|
| **1. Core** | The combat-idle signal, the Milestone engine, **Contest with Riposte**, **Endure with Catch Breath** (Close Call only), Match-local state, minimal HUD, telemetry | **Now**, on the current Match | A Stage 1 run test shows the Lanes are pursued and completed, Riposte and Catch Breath are used, and no unanticipated exploit. If Milestones don't change play, **reconsider the system before expanding.** |
| **2. Arena and PvE integration** | Pressured Exit, Warden damage as an Endure source, the Explore feats (kept **inactive** until its reward exists) | **After Tickets 21 to 24 are built** | Stage 1 evidence, plus the integration contracts below verified in code |
| **3. Reserve** | The full Reserve spec below | **Only on evidence**, independent of whether Riposte and Catch Breath succeeded | Loot-saturation evidence (see below). If it proves valuable independently of Milestones, **revisit whether it belongs in the base Kit system.** |

#### Scope

- **Stage 1:** `MilestoneRules` (pure: Lane state, feat detectors, the activation rule), `Milestones` (server: state, events, unlock, banners, telemetry), the shared combat-idle signal, Riposte and Catch Breath (rules, server state, `Combat` integration, Protocol, visuals with placeholder audio), the Contest and Close Call feats, HUD indicators, `Tuning.milestones`, specs, the Studio matrix, the `CONTEXT.md` terms, and the doc updates in this commit.
- **Stage 2:** Pressured Exit, Warden and Riftling sources, the Explore feats, and the integration hooks in Tickets 21 to 24.
- **Stage 3:** Reserve in `KitRules`, `Kits`, `Pickups`, `Match` (elimination), client copies, HUD and specs.

#### Non-goals

- XP, levels, currency, Relics, Equipment, permanent progression or cosmetics. A reward menu, a token bank, a bench or a shared Tool slot (all designed, then dropped).
- Any reward that raises damage, Health, movement speed or Evade uptime, or that changes Ability Rank or Pickup supply.
- Lobby-size scaling of feats, collusion detection or punishment, a Dais-specific exclusion, or extra combat restrictions without evidence.
- A generic Lane framework. The gates are flat booleans.
- Stagger, Knockback or forced movement, projectile or PvE parrying, knockback resistance.
- Final art, audio and animation. **No upload without explicit approval**: placeholder built-in sounds and simple Highlight-style visuals only.
- Changing the 24-drop Cache economy, `LootRules.validate`, Last Hit loot ownership, Ability Ranks, or the shared Match `random` draw order.
- Fixing the recorded discrepancies below.

#### Dependencies

**1. Requires Tickets 21 to 24 to be built first (Stage 2 only):**
- Ticket 21: a per-player start Shard and each Riftling Home's Shard.
- Ticket 22: a read-only **Section at a position**, the Warning flag, and the **applied** removal time and settle window (`Collapse.sync`). Pressured Exit uses the applied time, so a stalled Collapse shifts it consistently.
- Ticket 23: the Warden's Engagement lifecycle and a per-player effective-damage tally.
- Ticket 24: Riftling kills with Last Hit credit.

**2. Ticket 25 builds itself (Stage 1, testable on the current Match):** the combat-idle signal (stamp points in `Combat`, `Casts` and Charging, with PvE stamps added by Tickets 23 and 24), the Milestone engine, Riposte, Catch Breath, the player-damage window keyed by a **source class** so the Warden and PvE plug in later, input actions, HUD, telemetry and specs.

**3. Integration hooks added to earlier tickets (requirements, not redesign), recorded in each ticket:**
- **Ticket 21:** expose each player's start Shard and a Shard-membership lookup.
- **Ticket 22:** expose a Section lookup from a position, the Warning flag and the applied removal and settle times, read-only.
- **Ticket 23:** keep a **per-player effective-damage tally per Engagement**, cleared when regeneration starts, on a full reset or on despawn, and report a `sourceClass` on its hits. Last Hit still decides loot.
- **Ticket 24:** report a Riftling kill with its killer and Home Shard, report `sourceClass` on its hits, and give no credit on despawn, Fault or Return.

**Stage 3 additionally relies on** Ticket 21's and Ticket 22's Kit-drop spread and live-floor fallback, which currently assume the drop spread of an existing Kit. They must handle **up to three** items.

#### Feature gates

`Tuning.milestones` holds flat booleans, not a Lane framework:

| Gate | Stage 1 default | Rule |
|---|---|---|
| `contest` | on | |
| `endure` | on | Close Call only until `endurePressuredExit` |
| `endurePressuredExit` | off | Requires Ticket 22 built |
| `explore` | **off** | **Requires `reserve`.** A spec asserts `explore` implies `reserve`, so Explore is never active without its reward. |
| `reserve` | **off** | Stage 3 |

Milestones exist only inside a Match, which is itself dormant under the Reference Kit. With every gate off, or outside a Match, there are no Milestone attributes, tags, remotes or log lines, and the arena, dummies and Stage 3 builds behave as today. **No placeholder reward is introduced for an inactive Lane.**

#### Settle before building Stage 1

- **Input.** Three new actions (Riposte, Catch Breath, and Stage 3's Reserve swap) are reviewed together for keys and accessibility. Keyboard is the only input today (no gamepad or touch handling exists). The actions are device-independent (`Riposte`, `Recover`, `SwapReserve(slot)`) and **no key is assigned here**. Studio tests use placeholder bindings only.
- **The combat-idle window** and the other values below are placeholders until measured.

#### Milestone engine

- **State** is per player and per Match: for each Lane, `earnedAt`, `unlockedAt`, uses spent. It is created at GO, cleared at Match end and at Elimination, and never persists. There is nothing to carry across a Match or a rejoin.
- **Completion** is immediate when a qualifying feat occurs, once per Lane, with a banner and a cue. A Lane that is gated off never completes.
- **Activation rule.** A reward is usable once **both** hold: at least **60 s** (`milestoneUnlockSeconds`) since **GO**, and the player has reached an out-of-combat moment **after earning it** (`unlockedAt` is the later of the two). **The calm requirement applies once, to the unlock.** It never revokes access later, so Riposte can be used in combat after it is unlocked. A reward earned while already calm unlocks as soon as the time gate allows.
- Eliminated players see no Milestone UI, and a Spectating player's earned rewards do not exist.
- **Hub and non-Match play:** no Milestone state.

#### The combat-idle signal

One server value, `lastHostileAt[player]`, and one rule: a player is **out of combat** when `now - lastHostileAt >= combatIdleSeconds` (about 3 s, tunable). It is the **same signal** for Catch Breath, the unlock rule and Stage 3's Reserve. It is checked **when an action is committed**, not only when it is opened.

**Stamped by** (server-authoritative, meaningful events only):
- An accepted hit dealt or taken, **including a fully absorbed hit**.
- A hit voided by Evade, **for both attacker and defender**, and a hit voided by Parry, for both.
- The player's own damaging Cast, Basic swing or Charge start.
- PvE damage dealt or taken (stamps added by Tickets 23 and 24).
- A Riposte activation.

**Never stamped by:** a rejected or invalid request, a non-hostile Cast (Stone Guard), or proximity. A missed projectile that passes without hitting is **not detected**, so a missed attack doesn't keep a target in combat indefinitely. This is accepted.

The existing 7 s `fightIdleSeconds` (fight logging) and the 7 s Natural Recovery delay are **separate** and are not reused.

#### Contest (Stage 1)

Completed by **any one** of the following. Each path is independent.

| Path | Rule |
|---|---|
| **Damage** | About **60 effective damage** to players, **summed across opponents**. Effective means Health actually lost after Shield, with no overkill, from accepted hits on living Match participants. PvE, Collapse and self-damage never count. A Reaction's damage is credited to whoever triggered it. |
| **Elimination** | Eliminating a player, by the Last Hit rule (a Void or Collapse death with no player credit doesn't count) |
| **Reaction** | Triggering a Reaction against another player (the `Triggered By` credit) |

Reward: **Riposte**.

#### Endure (Stage 1 for Close Call, Stage 2 for Pressured Exit)

Completed by **either** path. **Only damage from players and Wardens qualifies.** Riftling and Sentry damage does not, and a Warden contributes only once Ticket 23 is built. Damage is **effective Health lost**, not raw incoming damage. The qualifying tally keeps `(time, effective damage, sourceClass)` entries for 10 s.

| Path | Rule |
|---|---|
| **Close Call** | After a qualifying hit, Health is about **30 or below**, and the player is alive **10 s** after the last qualifying damage. The timer resets on each qualifying hit, and the player must stay alive throughout. |
| **Pressured Exit** | About **15 effective damage** in the 10 s before a Section's **applied** removal time, taken while standing on that Section **during its Warning**, and the player is alive on remaining ground after the removal **and the settle window**. |

Reward: **Catch Breath**. Pressured Exit's timing and damage window are server-authoritative.

#### Explore (Stage 2, inactive until Reserve exists)

Completed by **either**:
- **Kill a Riftling on another Shard:** the player has the Last Hit, and the Riftling's Home Shard is not the player's start Shard. A Riftling consumed by the Collapse, a Fault or a Return gives no credit.
- **Contribute about 25% of a Warden's maximum Health** as effective damage during **one Engagement**. Counters clear when the Warden's regeneration starts, on a full reset, or on despawn, **not** at the start of its walk home. Credit comes the moment the threshold is reached, and later death, despawn or someone else's kill does not undo it. Last Hit still owns the Warden's loot.

Reward: **Reserve**. Explore is an **early-to-mid-Match opportunity, not a guaranteed Milestone.** Its availability falls as PvE actors die and the Arena collapses, some players will never earn it, and that is intentional and measured.

#### Riposte (Stage 1)

An activated tool, **3 fixed uses per Match, no refunds**. A use is spent on activation. A rejected activation consumes nothing.

- **Stance:** about 0.1 s startup, then an active window of about **0.35 s**. The player can't Cast or Basic during the stance. A Dash ends it.
- **Eligible hits:** a **Sector** hit from a **player** (Basic Attack, Arc Burst) that lands inside the window, from the front of the defender. **Not** projectiles (Tidal Bolt, Rift Hook, Sentry shots) and **not PvE attacks.**
- **Order in `Combat`:** Evade is checked first, then Parry, then Shield and the rest. A Parried hit has the **same suppression as an Evaded hit**: no damage, Status, Reaction, Knockback or Kill Recovery. The result kind is `Parried`.
- **Success:** the stance ends, the defender's **Basic cooldown refreshes**, and the **next Basic within 1 s** has a range of about **9.5** studs instead of 8, using the existing Basic validation. No damage bonus, no stagger and no forced movement. 11 studs is a **tuning ceiling, not a starting value**.
- **Failure:** if the window ends with no parry (including after a Dash), about **0.4 s** with no Basic or Cast. Dash stays available.
- **Refused while:** not unlocked, no uses left, Charging, Stunned or Frozen, already in a stance or recovery, or outside an active Match.
- **Assembly defaults, not individually grilled:** a stance parries **only the first** eligible hit and then ends, so two attackers can't both be negated in one window. A multi-target Arc Burst is negated only for the parrying player. A parried attack's cooldown is still spent.
- **Authority:** under ADR-0002 facing and movement are client-controlled. The server reads root position and orientation at the check, with a stated tolerance. A client could spin to face an attacker, so the 0.35 s window and the 3 uses are the real limit.
- The **reach override** has to reach the server's hit spec and the client's drawn Sector (`CosmeticSectors`). What `CastStarted` carries is to be verified when building.

#### Catch Breath (Stage 1)

An activated tool, **2 uses per Match**.

- **Channel:** about **1.5 s**, stationary. **About 30 Health is restored only on successful completion** (clamped to maximum). The checkpoint variant (half at the midpoint) is recorded for use if late interruptions prove too frustrating.
- **Interrupted by:** movement (a server-checked displacement over a small tolerance from the channel start), a Dash, any Basic, Cast or Charge start, or any accepted hit, **including fully absorbed damage**. **An interrupted channel consumes the use.** A rejected activation (not a channel) consumes nothing.
- **Refused while:** in combat (the shared signal), Charging, Stunned or Frozen, not unlocked, no uses left, or outside an active Match.
- **Server-authoritative timing and cancellation.** A visible aura and an audible cue let nearby opponents see the channel.
- Movement Validation's 0.5 s sampling is too coarse for a 1.5 s channel, so a separate server check is needed. It does not change Movement Validation.

#### Reserve (Stage 3, specified, not to be built without evidence)

The complete approved design is kept here so it isn't lost. **Nothing in Stage 1 or 2 builds any part of it.**

**What it is.** One extra Ability held in storage. It is **not castable**. It is not a third Ability slot, and Q and E remain the only castable slots.

**Entry (displacement only).** When a Pickup replaces a slot **out of combat** and the Reserve is empty, the **displaced Ability goes to the Reserve** instead of dropping. In combat, replacement works **exactly as today** and the displaced Ability drops. With the Reserve full, it drops as today. There is **no automatic collection into the Reserve and no discard action.** To lose a Reserve Ability, swap it into a slot and replace that slot.

**Reserve is inert in combat:** no Ability enters, leaves or swaps with it while the player is in combat. Rank upgrades to a stored Ability are **allowed** in combat, because they change nothing active.

**Swap.** A **Reserve modifier + Q or E** (key undecided, settled in the input review) exchanges the Reserve with that slot, or fills an empty slot. **F + Q or E stays exclusively for Pickup replacement**, and the modifier alone does nothing. The swap is a separate request (`SwapReserve(slot)`), not a replace request. It is out of combat only, validated at commit time. The incoming Ability gets the existing **0.85 s Equip Lock**. A rejected swap consumes nothing, never drops an Ability, doesn't touch cooldowns or Cast state, and shows brief feedback if it was rejected for combat. It uses the existing replacement rate guard (burst 5, 5 per second).

**Ownership and Rank.** One copy of each Ability across Q, E and the Reserve, enforced by a separate `KitRules.owns` lookup. **`KitRules.find` stays slot-only**, so a Reserve Ability can never be cast, on server or client. Rank travels with the Ability, and the cap stays 2. **Cooldowns belong to the Ability and keep running while it is stored** (an existing invariant). A duplicate Pickup **upgrades whichever copy you own**, equipped or stored, by the existing formula, with no Equip Lock. At the cap, the Pickup stays in the world. Auto-collection works for an eligible Reserve upgrade.

**Elimination.** `Kits.takeAll` hands over Q, E and then the Reserve **atomically**, each dropped as its own Pickup at its Rank, and clears the Reserve attributes in the same step. The same rules apply to Eliminations, leaving and Void deaths, including a missing character. Three Pickups need three spread positions, which must stay individually collectible and pass the Kit-drop validation of Tickets 21 and 22. **Verify the three-Pickup spread in Studio before fixing coordinates.**

**Replication and UI.** New attributes (`KitReserveAbility`, `KitReserveRank`), copied into the client's `Casting` and `Replacing` modules (which must ignore the Reserve for casting), a small Reserve icon in the HUD, and while the modifier is held, a preview of the Reserve Ability and the available targets.

**Telemetry:** Reserve stores, swaps and upgrades.

**Tests (Stage 3, blocking):** one copy of each Ability across all three positions, a Reserve Ability can't be Cast, combat upgrades don't permit swapping, no swap in combat, atomic drops including missing-character and disconnect cases, a duplicate upgrades the Reserve copy, and `takeAll` ordering and attribute clearing.

**Known confounds and risks:** it is a half-step toward a third slot, which the docs defer until about 6 to 8 Abilities (it would confound that test), it makes Rank 2 more common (three owned Abilities give more duplicates somewhere to go), and an early advantage for a PvE-first player. These are measured, not assumed harmless.

#### Protocol, authority and guards

- New requests: `RiposteRequest` and `RecoverRequest` (Stage 1), `SwapReserveRequest` (Stage 3). Each is a minimal, validated payload, and **declares its guard**. Following Ticket 20's inventory, only `ReplaceRequest` is rate-guarded today and the Cast, Charge and Dash requests rely on state gates. Riposte and Catch Breath rely on state gates (uses, stance, recovery, unlock), `SwapReserve` reuses the replacement bucket. **Each new remote throttles its malformed-request warning** to one per second per player.
- Results travel as a `Riposting` tag (like `Evading`), a `Recovering` attribute with its start and end, a `Parried` `CombatResult` kind, and a Milestone event for banners. `Feedback` must treat the new kind explicitly.
- **Server authority.** The server owns every state, use count, unlock time and validation. Clients display only.
- **Hit reports** need a `sourceClass` (Player, Warden, Riftling, Sentry) so Endure can filter. Player hits get one in Stage 1.

#### Presentation

Minimal: a Lane banner on completion (naming the feat), a "ready" notice when a reward unlocks, two small indicators near the Dash charges (Riposte uses and state, Catch Breath uses and state), the Riposte stance and parry visuals and cue, and the Catch Breath aura and cue. Placeholder built-in sounds and Highlight-style visuals only, with no new asset upload. Eliminated and Spectating players see none of it.

#### Telemetry

`[Riftfall][Milestone]` lines carrying `match=` and `t=` (seconds since GO), with a capped event count, and failure contained and never blocking play.

- **Earned and unlocked:** Lane, path, source, lobby size, `earnedAt`, `usableAt`. Distinguish rewards earned but never activated from rewards activated but never used.
- **Riposte:** activations, successes, failures, **prevented damage** (the damage the hit would have dealt before Shield, including a Reaction bonus that would have applied), the unused uses at the end, **by opponent Kit**, and when each use was spent.
- **Catch Breath:** completions, interruptions by cause, **actual Health restored**, uses, and timing.
- **Endure:** the **source of each completion** (player or Warden).
- **A trade flag:** two players' Contest completions within about 5 s of each other from damage against each other with no Elimination in that fight. This marks a suspected pair and **punishes nothing**.
- **Time-to-all-three** by lobby size and Match outcome, and the rewards held when entering the Dais.
- **`MatchResult` additions:** the Milestones earned. This line has fixed fields, so any parser is updated with it.
- **Reserve demand signals, recorded in Stage 1 so the Stage 3 decision has evidence:** replacements, Abilities dropped by replacement, and Pickups left in the world at the Match end.

#### Tests

All blocking specs run in `RunAll` on the Server in Play, and every existing spec stays green (427 at version 98).

- **S1 `MilestoneRules`:** each Lane's feats, one completion per Lane, the 60 s and calm unlock (the later of the two, and not revoked), and clear on Elimination and Match end.
- **S2 Contest:** the damage path counts effective damage only (Shield, overkill, PvE, Collapse and self excluded, summed across opponents), the Elimination path, and the Reaction path credited to the trigger.
- **S3 Endure:** Close Call (qualifying hit, 30 or below, the timer reset, death resets, non-qualifying sources ignored), and Pressured Exit's window, Section and settle rules once Ticket 22 exists.
- **S4 combat-idle signal:** every stamp source, both sides of an Evade and a Parry, rejected requests don't stamp, the window, and commit-time checking.
- **S5 Riposte rules:** the stance timings, first hit only, Sector and player only, no projectile or PvE parry, the order against Evade and Shield, the refresh and reach override for 1 s, the failure recovery, 3 fixed uses with no refund, and every refusal consuming nothing.
- **S6 Catch Breath rules:** completion heals once and clamps, every interruption consumes the use, rejected activations don't, and the combat refusal.
- **S7 Protocol:** each new request's validation, malformed payloads, throttled warnings, and the `Parried` kind.
- **S8 dormancy:** no Milestone state, attributes, tags, remotes or log lines under the Reference Kit, with the gates off, or outside a Match. The `explore` implies `reserve` assertion.
- **S9 elimination:** Milestone state clears at the defined point of the Elimination order.

#### Studio matrix (Stage 1, by `studio-tester`, two clients)

- **M1 dormancy:** the Reference Kit Stage 3 arena is unchanged.
- **M2 Contest:** each of the three paths, with a real second client.
- **M3 Close Call.**
- **M4 unlock:** a Lane completed in a fight unlocks only after the calm moment, and never before 60 s.
- **M5 Riposte:** a success on a Basic and on an Arc Burst, a failure, a failed Riposte against a projectile, the order against Evade, a PvE attack not parried, and 3 uses.
- **M6 reply:** the 9.5 reach applies once, within 1 s.
- **M7 Catch Breath:** a completed channel and each interruption type.
- **M8 forged and flooded requests,** from a second client's real remotes: unlocked, no uses, wrong state, malformed payloads, and bursts.
- **M9 two clients:** state, banners and cues replicate and clear correctly across a Match and an Elimination.
- **M10 telemetry:** the lines and `MatchResult` fields.

#### Evidence needed before each stage

- **Stage 1 to Stage 2:** the Lanes are pursued and completed at plausible rates, Riposte and Catch Breath are used, nothing dominates by a wide margin in the telemetry, and no new exploit appears. **If the system doesn't improve gameplay, reconsider it before expanding.**
- **Stage 2:** Tickets 21 to 24 built and committed, and the integration hooks verified in code, including Sentry and Warden attribution on hit reports.
- **Stage 3 (independent of Stage 1's rewards):** loot saturation: full-Kit players leaving Pickups unused, how often a replacement drops a useful Ability, time to a full Kit, and Rank progress. Decide whether Reserve belongs in the base Kit system instead. Also settle the confound with the two-versus-three slots test.

#### Recorded risks

- **Explore** may be too easy in 2-player Matches, or unavailable too quickly in full lobbies. Reserve may give an early advantage to a PvE-first player, and players may feel forced to pursue Explore before PvE actors disappear. No replacement feats or lobby scaling without run-test evidence.
- **Riposte** may be too strong against Basic and Arc and useless against projectile Kits, so the matchup difference is measured by opponent Kit. Its **prevented damage** is measured, not only its success rate. It sits close to Evade in kind.
- **Total defensive value** (three Ripostes and two Catch Breaths) is large even though it can't be used at once. Hoarding for the Last Stand is a tension, not a flaw.
- **Collusion:** two consenting players can trade damage to complete Contest and Endure in about 25 s, and Natural Recovery makes that nearly free. **This is a known, accepted risk, not a solved one**, and Catch Breath isn't balanced around it.
- **Endure** may rarely complete in short Matches. If it rarely contributes, reconsider the Lane, don't make its feats easier. If it is solved through PvE, the first lever is excluding PvE damage from the remaining routes.
- **Stage 1's pacing evidence** will come from the current Match map. Ticket 20 measured the old map (about 7.8 s to a first Ability and about 17 s to a full Kit, floor pacing), and the Rift Crown establishes a new baseline.

#### Completion criteria

**Stage 1:**
- All blocking specs green in `RunAll`, with every earlier spec still green.
- The blocking Studio matrix green.
- Dormant under the Reference Kit and with the gates off, verified.
- Docs and results recorded in this ticket and the quirks doc updated, with every TEMPORARY hook removed (grep the word).
- A clean commit.

**Stage 2 and 3:** their own completion criteria are written when each is approved, from this section.

**Publish checkpoint** (separate manual step): the developer publishes and the version is recorded in [Checkpoints](#checkpoints).

#### Future work

Recorded, none built: a second reward per Lane (Echo, Backtrace, Scout's Sweep, Survey, Cache Cracker, Rift Seals, Feint, Heavy Hand, Read the Kit, Wager, Exposed Opener, Danger Sense, checkpoint healing), an Evade-based Endure path once run-test evidence supports it, and Reserve as a base Kit-system decision.

#### Recorded discrepancies (not changed by this ticket)

| # | Discrepancy | Where |
|---|---|---|
| D1 | Resolved by Ticket 21's build (2026-11-11): the loot comment now says 6 each of 24 | `Tuning.loot` |
| D2 | `Tuning.match.maxSeconds` is 480, where Ticket 22 sets a 600 s failsafe | `Tuning.match` |
| D3 | Resolved by Ticket 20: `startingKit` was a TEMPORARY `"Empty"` and is `"Reference"` again at version 98 | `Tuning.kit` |

Ticket 25's timings assume Ticket 22's schedule for Pressured Exit only. Stage 1 doesn't depend on D2.

#### Assembly defaults (not individually grilled, recorded so they are reviewable)

- **Riposte:** a stance parries only the first eligible hit and then ends. The stance blocks the player's own Cast and Basic. A Dash ends it, and the failure recovery still applies. Stunned and Frozen refuse an activation (a Rooted player may act). A parried multi-target Arc Burst is negated only for the parrying player.
- **Catch Breath:** a Basic, Cast or Charge start interrupts the channel as well as movement, a Dash and a hit. The movement tolerance is a small server-checked displacement from the channel start (about 1 stud).
- **All tools:** refused while Charging, outside an active Match, and when not unlocked, with the refusal consuming nothing.
- **`milestoneUnlockSeconds` (60)** is measured from GO (the start of the Running phase), not from the Countdown.
- **`sourceClass` names:** Player, Warden, Riftling and Sentry. Only Player and Warden count toward Endure.
- **The Contest Elimination path** counts a ring-out Elimination that carries displacement credit (a Hook pull or a Tidal push into the void), because it names a player as the killer. A Void or Collapse death with no credit does not count.
- **Telemetry** is capped at about 32 Milestone events per Match, and a failure to log is contained.
- **Banner and cue text and visuals** are placeholders for the developer to review.
- **Scope:** Stage 1 is a large piece of work, roughly one and a half tickets. If it proves too large for one sitting, split it at **1A** (the signal, the engine, Contest and Riposte) and **1B** (Endure and Catch Breath), each with its own commit, with the Stage 1 evidence gate after 1B.

#### Review notes (2026-11-10)

An adversarial pass against the Match, Combat, Kit, Pickup, Collapse and PvE specifications found **no blocking contradiction**. What the review found and fixed in the documents:

- **Contribution wording:** `monsters.md` said contribution is "never used to split the reward". The Explore feat uses it, so the wording now says a Milestone is attribution, not a reward split, and the Last Hit still owns loot.
- **"Only upgrade and rarity system":** `ranks-and-pickups.md` now says the only **Ability** upgrade and rarity system.
- **"An inventory still doesn't belong":** `code-structure.md` now says the Reserve is one Kit field, not an inventory.
- **Mid-fight spike:** Contest completes at 60 damage, usually while ahead, so an immediate Riposte would be an in-fight spike. The unlock rule (the 60 s gate plus a calm moment) fixes it.
- **Parry order and multiples:** two attackers could be parried in one window, and the order against Evade and Shield was open. Both are now fixed in the Riposte rules.
- **A hazard in `KitRules.find`:** extending it to see the Reserve would make a Reserve Ability castable (`canCast` and the client use it). A separate `owns` lookup is specified instead.
- **Kit-drop spread:** Ticket 21's and 22's drop-spread validation assumes today's drop counts, so three items need a check (Stage 3 dependency).
- **Source filter attribution:** Endure needs a `sourceClass` on hit reports, which Tickets 23 and 24 must provide.
- **Far Shard** was dropped as trivially cheap. This also removed the Shard adjacency dependency.
- **Flood guards:** per Ticket 20's inventory, each new remote declares its guard and throttles its malformed warning.
- **`explore` without `reserve`:** the gate rule is asserted by a spec.

Accepted, with evidence to follow: the recorded risks above. Correctly deferred: see Future work. Review result: **ADR-0009 is warranted and written.** The other decisions are tunable gameplay and rules plus system work.

### 26. Rift Collapse Visuals and Atmosphere

**Status: Specified, Not Built. Specified 2026-11-10 and reviewed adversarially the same day.** The ticket is **staged** behind a metadata prerequisite: Stage 1 starts only after Tickets 21 and 22 are built, and the developer approves every stage before the next begins. Every design point was grilled with the developer and settled. The reasoning, rejected options and edge cases are in [`collapse-visuals-design.md`](collapse-visuals-design.md); this entry is the build contract. Exact module names and folder layout are provisional and recorded when built. Every value carries one of these labels:

- **fixed:** an approved requirement. Changing it is a design change and is recorded here.
- **tunable:** a starting value that can change from evidence or the art pass without a redesign.
- **check:** an implementation verification, performed and recorded at build. The result picks a named fallback and never invents a fact in advance.
- **deferred:** not built here.

**Expected values and art quality are not pass criteria.** Luminance, contrast, haze, hue and frame-time figures, audio balance and "feels like the Unmaking" produce tuning notes. Only the **Completion criteria** decide each stage.

#### Summary

Ticket 26 gives the Rift Collapse its presentation, **the Unmaking**: the arena is being **erased**, not burned or darkened. The world loses colour and substance as layers are consumed, each Warning Section visibly destabilises, each removal is an **Unravel**, and the Dais ends as the last surviving piece of reality. **Disappearing ground remains the only Collapse hazard.** Everything here is client-side, cosmetic, non-interactive and derived from state [Ticket 22](#22-rift-collapse) already replicates. It adds **no gameplay change, no new RemoteEvent and no schedule change**, and no effect may obscure PvP information.

#### Stages and gates

| Stage | Contents | Can be built | Stops for |
|---|---|---|---|
| **Prerequisite** | The authored presentation metadata and its additive validator checks (below) | With the [Ticket 21](#21-rift-crown-arena-layout) build, or as the first step of Stage 1 | Nothing separate |
| **1. Readable Collapse** | The scoped client controller and lifecycle, the Warning vocabulary, the Unravel, the distant pulse, the basic stage presets, Full and Lite tiers, placeholder Collapse audio, the Results hold and the Hub restore, regression against Tickets 21 and 22 | After Tickets 21 and 22 are built | The developer's go-ahead. **Stage 1 alone is a complete, playable Collapse.** |
| **2. The Unmaking atmosphere** | The procedural horizon, damage-recording far-field fragments, dust, the ambience bed, positional hums, the audio identity pass, the full capture matrix and the identity checklist | After Stage 1's Studio validation and go-ahead | The developer's go-ahead and visual identity sign-off |
| **3. The Last Stand** | The Last Anchor light, the final draining sound, the Last Stand ambience, six-player Dais readability, Layer 4 enabled and disabled validation | After Stage 2's go-ahead | The developer's go-ahead |

Pure rules and their specs touch nothing in the map and may be built early. Everything else waits for Tickets 21 and 22.

#### Scope

- `CollapseVfxRules` (pure, provisional name) and a client `CollapseVfx` controller (provisional name).
- **Stage 1 replaces or extends Ticket 22's minimal client telegraph and placeholder cues in place.** No second controller, no duplicate warning effects and no duplicate audio. The module names are not given in Ticket 22, so the first step is a **check** that finds them.
- The authored metadata, the effect budgets and tiers, the stage presets, the audio groups and levels, the diagnostics, specs, matrices and release gates.
- `CONTEXT.md` terms, and the amendment notes on Tickets 21 and 22.

#### Non-goals (fixed)

- **Any Rift hazard:** damage, slowing, blinding or pushing volumes, fog gameplay, an advancing wall, an airborne Rift volume. Fog and storm are atmosphere only.
- **Any change to Ticket 22's gameplay contracts:** the schedule, atomic and phase-locked removal, collision, the removed-geometry invariant, the settle window, server-authoritative timing, or Layer 4 being optional.
- **Pre-Warning cues** of any kind on a Section, Shard or place, per-Shard foreshadowing, ghost geometry, Riftlight scars and floor-level afterimages.
- **Camera movement or FOV changes** (shake, kicks), screen flashes, vignettes, and full-screen audio-linked effects. A very short FOV or vignette dip is a named fallback, **not built** without separate approval.
- A **permanent exposed-edge trim.** Ticket 21's authored edge treatment is used first, and any trim needs test evidence and approval.
- **Uploads and new assets:** textures, meshes, skyboxes, sounds. Placeholder built-in sounds first. **No upload without explicit approval.**
- Adaptive music, a ducking system, an adaptive quality manager, startup benchmarking, a third tier, a runtime colour registry or hue validator, new RemoteEvents, client-to-server telemetry or an analytics backend, and a Last Stand feature flag.
- Concurrent-Match infrastructure (see Lifecycle), a Hub restyle, and changes to Tickets 23 to 25.

#### Dependencies and amendments

- **Ticket 21 (amended, presentation metadata only).** See [Presentation metadata for Ticket 26](#presentation-metadata-for-ticket-26-added-2026-11-10). This is the only change to Ticket 21, and it adds no gameplay scope.
- **Ticket 22 (relied on, unchanged).**
  - **Replicated state used:** the Workspace attributes `CollapseLayer`, `CollapseWarning`, `CollapseWarningAt` and `CollapseRemovesAt`, the per-Section `CollapseState` and `CollapseRemovesAt`, and the existing `MatchId`, `MatchPhase` and per-player `MatchState`. The authored Section `CollapseLayer` must also be readable by clients (**check**).
  - **No new `CollapseState` transition** unless testing proves the existing signals insufficient.
  - Ticket 22's criterion 9 and its presentation, spectator and six-roster matrix rows must still pass after the replacement.
  - The Ticket 22 cue semantics stay: a global sting and announcement at 30 s, the Urgent presentation at 10 s, the local per-second cue **only for players on Warning ground** in the last 5 s, a positional removal cue, and no global per-second ticks.
  - Ticket 22's local alert, arrow and HUD line are not changed.
- **Ticket 23 (Warden).** Warden telegraphs use hostile red as filled wedges and discs. A Collapse removal destroys a Warden at once, and the Unravel only draws cosmetics at cached bounds.
- **Ticket 24 (Riftling).** Violet-teal seams on a moving body are the creature's own look.
- **Ticket 25 (Milestones).** Its Feedback sounds and cues must stay above Collapse audio in the mix. Its Highlight-style visuals count against any Highlight budget (see the outline fallback).
- **Ticket 27 (Minimap, specified 2026-11-11, added to this ticket by amendment).** The Minimap and Tactical map are a **consumer** of this ticket's shared `matchId`-keyed Collapse accessor and its Participant, Spectator and None context derivation, **when they exist**. This ticket stays an **optional integration dependency, never a prerequisite**: Ticket 27 can be built on Ticket 22's replicated attributes alone, and it adopts the accessor if this ticket lands first, with no second Collapse model. This ticket changes no signal that [Ticket 27](#27-minimap-and-objective-awareness) reads (`CollapseState` on each Section and connector model, `CollapseWarning`, `CollapseLayer`, `MatchId`, `MatchPhase`), and it must not make the Minimap's Warning fill or the world telegraph disagree.

#### Identity and colour roles (fixed unless marked)

- **Start:** a cold, muted stone world. **Escalation:** progressive desaturation and bleaching of sky and distance, hardest in the final stages. **Void:** pale, never black. **No literal white-out:** dark stone and structural contrast are preserved, so players, Abilities, the HUD and lethal edges stay readable. Escalation must feel dramatic and must not be flattened for contrast.
- **Riftlight** is rose-magenta in about the 315° to 330° hue band (**tunable**, exact values in the art pass). It means ground instability and reality erasure, never combat damage. It appears in **exactly four places:**
  1. static baseline seams
  2. the Warning vocabulary on Warning Sections
  3. the Unravel flash and fragments
  4. the non-spatial horizon (Stage 2), which never uses the Warning rhythm, because the same rhythm must always mean the same deadline

  It is never ambient fog.
- **Roles:** hostile red means an incoming enemy attack. Ability colours (Tidal blue, Arc Burst yellow-white, Hook and Stone Guard amber and grey) mean Abilities and Pickups. Violet-teal is the Riftling. Riftlight is reserved as environmental identity. A future Ability may use related hues only if its shape and meaning stay clearly distinct and explicit visual testing passes.
- **Priority ladder (fixed):** HUD, then players, Ability effects and Pickups, then ground warnings, then the Unravel, then atmosphere and horizon. No Riftlight effect exceeds the peak luminance of an Ability effect it overlaps. A Warden attack overlapping a Collapse warning stays immediately recognisable, and the warning must still read as ground disappearing. Neither erases the other.
- **Hue separation** of at least about 40° from Ability and Pickup colours is an **initial art target**, not a pass rule. Separation from hostile red rests on shape, placement, motion and luminance, with the hue gap as extra margin.

#### Presentation metadata (prerequisite, fixed)

- Each Section and connector carries an ordered **`CutLine`** set of Attachments along its inward breaking boundary, with the fewest points that represent the boundary accurately.
- Each Section has a **`VfxGroup`** attribute, defaulting to `ShardId` unless the authored layout needs different grouping (rim links and connectors are assigned during review, **check**).
- There is one **`BeaconAnchor`** per `VfxGroup` **per `CollapseLayer`**, because a Shard spans layers.
- Anchors are **generated once at authoring time** from authored geometry, then reviewed for irregular boundaries. They are not runtime sampling.
- The structural validator gains **additive** checks: required anchors exist, and `CutLine` points lie on or near the Section's boundary.
- **Fallback:** a missing anchor logs once. A missing `CutLine` **omits** the cut-line, and a guessed boundary is never drawn. Fractures, HUD and audio are preserved. A missing **required** anchor fails validation before release.
- No runtime geometry sampling or boundary raycasts. Fracture start points use the floor samples Ticket 22 already permits.

#### Controller architecture (fixed)

- A client-only, **state-derived and edge-detected** controller. It uses attribute and ancestry signals, and its per-frame work is limited to effects that animate (the pulse and fracture growth during an active Warning). Tier, pulse phase and growth are pure functions of the server clock and replicated timestamps.
- **Context** is Participant, Spectator (including a Hub spectator) or None. The controller reads Collapse state through **one `matchId`-keyed accessor**. Atmosphere, global audio and the anchor light apply only in Participant and Spectator contexts. World effects and positional audio exist only inside a scope, so a None client (the Hub, waiting, another Match) gets no Collapse presentation, which matches Ticket 22's status-only Hub viewers.
- **One scope** per (`matchId`, role) owns every tween (created through one helper), pooled sound, effect object, connection, delayed callback and Lighting claim. Delayed callbacks check a scope epoch and do nothing after teardown.
- **Stage** is the number of authored layers with **no Sections present**, derived from the world, never from `CollapseLayer = 0`. It is derived **only while Running** and **freezes on leaving Running**, so the server's restore at Results end cannot drop the Results look. Last Stand means Layer 4 is consumed, so with Layer 4 disabled the maximum stage is 3.
- **Registration** is event-driven by Section appearance, so late replication is handled. Section bounds are cached on registration, before removal.
- **Unravel trigger:** a registered Section leaving its home parent while the Match is Running, once per `(matchId, SectionId)`. Never on restore, never for a Section the client never saw (a late joiner), never before removal. A coalescing window (about 100 ms, **tunable**) batches a layer's removals and dedupes the per-layer pulse. The holding-folder ancestry signal is a **check**.
- **Recovery points** (no per-frame reconcile): scope start, `MatchId` change, either Warning attribute changing (an order-independent, idempotent ensure), and after an effect-family error. Out-of-order or delayed observations converge. A late joiner or spectator computes the current target directly and never replays an earlier stage or pulse.
- **Errors:** protected boundaries around lifecycle operations and effect-family handlers only, with no defensive wrapping of per-frame updates. A family that errors three times is disabled for that Match. A fatal failure tears the scope down and restores Lighting. Cosmetic failures never affect Collapse, movement, combat or Match progression.
- **Lighting ownership:** a named list of properties and effects Ticket 26 controls, using its own effect instances wherever possible. Restore is **compare-and-restore**, putting a property back only if it still holds Ticket 26's last write. Any other writer gets an explicit coordination rule (**check**: find every other writer first).
- Effect parts are `CanCollide`, `CanQuery` and `CanTouch` false, outside the authored Section folders, and never move collidable geometry or touch gameplay queries.

#### Warning vocabulary (Stage 1)

Keyed by `CollapseWarning` and the per-Section `CollapseState`, never by a timestamp alone. Present **only** on Warning Sections.

- **Fractures:** branching Riftlight veins across the Section, growing in over the first few seconds (**tunable**, about 5 s) as `f(now - warningAt)`.
- **Cut-lines:** a jagged, broken line along the `CutLine` anchors, thin (about 0.5 stud or less). It marks where ground ends and draws no path to safety.
- **Beacons:** a thin, translucent vertical column per `VfxGroup` per layer, identifiable from another Shard.
- **Pulse:** phase-locked to `CollapseRemovesAt`, with peaks on **whole seconds before removal** and an identical phase for every Section sharing a deadline. Initial rates are **1 Hz, 2 Hz and 2 Hz** for the Early, Urgent and Final tiers (**tunable**), and **3 Hz is the validated ceiling**.
- **Tiers** (settled by Ticket 22): Early 30 to 10 s, Urgent 10 to 5 s, Final 5 to 0 s. The Final tier intensifies fractures, cut-lines and pulse and **never overexposes the Section**, so silhouettes, Pickups and Ability telegraphs stay visible.
- Jagged fractures and vertical beacons distinguish Collapse warnings from smooth Ability telegraphs. **Baseline seams** are dim, static and uniform, and carry no timing.
- Warning ground looks unstable and its inward parent looks stable (Ticket 22). Remaining ground **never** gains wear or instability over the Match. Instability vocabulary is reserved for Warning Sections.
- Connectors are Sections for this purpose. Dash gaps have no geometry and need nothing.

#### The Unravel (Stage 1)

- Triggered only as above. The ground vanishes completely at frame 0, with no ghost slab.
- A thin **cut-line flash** (about 0.2 s, **tunable**) reuses the Warning cut-line parts and is mandatory for every Section, including during a simultaneous removal.
- **Fragments:** small (up to about 1.5 studs), fall and bleach toward the pale void, gone within about 1 s, kept at or below floor level and never in front of survivors. Cosmetic stagger up to about 0.4 s after removal is allowed for fragments and flash only, **never for ground**.
- A global fragment budget per removal event (below). Fragments degrade first.
- A **distant pulse** once per layer, global and non-spatial: a small luminance step (about 10% or less, **tunable**) and one low-frequency sound, so the Unravel reads from another Shard.
- The exposed edge reads as the new lethal edge by **reusing Ticket 21's authored edge treatment** (**check** that interior boundaries have it). A pale, static, non-Riftlight trim is added only if tests prove it necessary, and needs approval.

#### Atmosphere

- **Stage presets** for stages 0 to 4. Atmosphere and ColorCorrection are the primary levers, plus ambient colours. **Not used:** Blur, DepthOfField, SunRays and strong Bloom.
- Sky and distance bleach progressively while combat geometry keeps contrast. Haze is a capped backdrop, and nothing inside the combat area loses contrast.
- An active Warning ramps the atmosphere modestly toward the next stage over its 30 s, and removal releases it with the distant pulse before settling at the new baseline. Ramps run only while changing, with no continuous loop.
- **Stage 1** ships the presets and the ramp, with stage 4 as a plain preset.
- **Stage 2** adds the **procedural horizon**: capped, distant, restrained Riftlight flickers on irregular timing, far-field fragments that grow in count and size with each consumed layer, and sparse dust. It is non-spatial and truthful (it shows only what already happened). Lite has none.
- **Stage 3** adds the **Last Anchor** (below).
- Whether the downward view reads as a pale void depends on the sky and Atmosphere per lighting technology (**check**).

#### The Last Stand (Stage 3)

- It begins **only** after Layer 4 is actually consumed. With Layer 4 disabled it is **dormant by data**, with no flag and no schedule change, and the Inner Crown receives the normal highest stage.
- The standard Unravel and distant pulse accompany the Layer 4 removal, then the atmosphere eases over about 2 to 3 s (**tunable**) into the strongest permitted stage: the palest void within the readability limits and the strongest horizon.
- **The Last Anchor:** one client-created, anchored, invisible part at the Dais centre with one neutral cool-white light (not Riftlight or an Ability colour), Shadows off, a capped brightness and a range of up to about 45 studs (**tunable**). It exists only on clients at runtime, so the authored map and the validator are untouched.
- The Dais is **structurally and visually unchanged**: no Riftlight, runes, decals, mesh swaps or added geometry. No particles, beams or debris rise above floor level in a clear cylinder around it. Void motes appear only below the lip.
- **The light is an enhancement, not a dependency.** The contrast between dark surviving stone and the pale erased world must carry the identity if the light is disabled or ineffective.
- Intensity comes from contrast, scale and atmosphere, not extra brightness or particles. The 1.5 s settle window and all gameplay are unchanged.

#### Audio

- **Identity:** eerie, hollow and reality-draining, never generic explosions, earthquakes or fantasy thunder. The world is disappearing, not physically breaking apart.
- **Stage 1 (placeholder built-in sounds, replacing Ticket 22's):** global stings at 30 s and 10 s, the removal impact, the distant low-frequency thump, and the Ticket 22 on-ground final-5 s cue unchanged. A visible equivalent exists for every cue.
- **Stage 2:** the stage ambience bed, **positional hums** at the beacon groups (sustained, pitch-rising tones with **no discrete ticks**, so rhythmic ticks stay with the on-ground cue), and the identity pass.
- **Stage 3:** the final 2 to 3 s draining sound that follows the atmosphere transition, then a sparse Last Stand ambience.
- Collapse audio has its own sound group with a **fixed ceiling below the combat and Feedback groups** (Ticket 25's cues included). It stays in the low and narrow-band range so combat cues stay clear. No dynamic ducking.
- Two voices are reserved for the threshold stings. A late joiner hears no historical stings. Spectators hear the global cues and positional cues relative to their camera. Positional hums and impacts are grouped by `VfxGroup`, never per Section.
- Existing sound groups are a **check**. Any final sound set or upload needs the developer's approval.

#### Readability and accessibility

Numbers are **tunable starting values**, and actual readability under overlapping effects and colour-vision simulation decides acceptance. Blocking gameplay failures must still be fixed.

- The void and horizon's mean luminance is at most about 0.55 relative, and lit stone to void is at least about 2:1, at every stage.
- A reference high-contrast target keeps at least about 80% of its near-contrast at 100 studs, and at least about 60% at the longest in-arena sightline.
- HUD text is at least about 4.5:1 against its own backing plate, so it never depends on the sky (**check** the existing plates).
- **Colour is never the only channel.** Fractures, cut-lines, pulse pattern, beacons, HUD text and arrow, and audio all carry the meaning, and colour-vision simulation runs on the captures.
- **Photosensitivity:** pulses are at most 3 Hz, nothing flashes a large part of the screen, and the distant pulse is a small step once per layer. Configuration validation rejects any rate above 3 Hz.
- Everything above must hold with **Atmosphere and ColorCorrection disabled**. Atmosphere is garnish, and its absence never removes a warning. Whether Atmosphere renders on the lowest settings is a **check**.
- **Outline fallback (conditional, not built):** a subtle, non-through-wall character outline only if the worst-case capture shows a genuine visibility problem. It must first check the Highlight budget alongside Ticket 25's Highlight-style visuals.

#### Quality tiers and budgets (tunable, per layer event)

Two tiers, **Full** and **Lite**, chosen **once at Match start** from Roblox's graphics-quality signals where reliable, with Lite when the signal is unknown, and held for the Match. Touch input alone never means low-end. The exact APIs are a **check**. A forced-tier override exists for Studio testing. Warnings never overlap (Ticket 22's 75 s cadence), so the worst case is always one layer event, Layer 1.

| Item | Full | Lite |
|---|---|---|
| Warning effect parts | at most 250 | at most 120 |
| Fractures per Section | essentials floor of 4, scaled down to fit | same |
| Cut-lines | complete | complete |
| Beacons | at most 8 | at most 3 (the group you stand on, then the nearest) |
| Dust (Stage 2) | at most 2 emitters per beacon group | 0 |
| Unravel fragments (pooled emitters) | at most 150 from at most 6 | at most 40 |
| Horizon emitters (Stage 2) | at most 8 | 0 |
| Concurrent Collapse audio voices | at most 8 | at most 5 |
| Total Collapse effect instances | at most 400 | at most 200 |
| Pulse update rate (active Warning only) | at most 30 Hz | at most 20 Hz |

- **Essentials floor (fixed, never tier-dependent):** the minimum fractures, complete cut-lines, the phase-locked pulse, the Unravel flash, the HUD alert and arrow, the local final-5 s cue, and the two reserved threshold voices. Essentials always win over optional budgets.
- **Drop order under pressure:** dust, far-field, fragments, farthest beacons, the anchor light, the ambience bed.
- **Budget conflict.** If Ticket 21's Section count makes the fracture floor exceed Lite's budget, that is a **design conflict flagged and resolved before implementation**, preferring a cheaper fracture representation (thin parts, Beams or built-in textures, no upload, **check**) over a larger cap. It never silently becomes acceptable.
- Pools are prewarmed in Countdown, Warning parts spawn over up to 3 frames, and the Unravel flash reuses existing Warning parts.
- The numbers are initial engineering budgets, tuned in Studio and on a device. Blocking requirements are correctness, cleanup and essential visibility.

#### Lifecycle (fixed)

- **Match end:** all animation stops immediately (pulses, loops, hums, in-flight debris, warning effects). The final static atmosphere **holds through Results** with no pulses, removal animations or escalation, then eases to the Hub baseline when the player leaves the Match context. If Results is skipped or interrupted, teardown still restores everything.
- **Mid-Warning cancel:** `stop()` clears the Warning through the attributes, and teardown backstops it.
- **New Match:** a new `MatchId` creates a new scope, Sections register as they appear, and the atmosphere eases to stage 0 during Countdown. A Section reappearing is never an Unravel.
- Participant to Spectator in the same Match **keeps the scope**. Switching spectate target restarts nothing. Leaving to the Hub ends the scope and restores Lighting.
- **Single-Match limitation, documented:** the replicated Collapse state is global Workspace attributes, matching the Milestone 2 one-Match constraint. The `matchId` accessor is the only preparation for concurrent Matches. Per-session state is Ticket 22's deferred item and is not built here.

#### Diagnostics

Concise, event-driven console lines in the existing key=value style with the prefix `[Riftfall][CollapseVfx]`, each carrying `match=` and `t=`. There are no per-frame logs, no remotes and no backend.

`Tier` (with its source: forced, quality or fallback), `ScopeStart`, `ScopeEnd` (with leftover instance, connection and tween counts, which must be 0), `Stage`, `Warning` (parts, fractures, beacons, capped), `Unravel` (sections, fragments, flashes, batch time), `Degrade`, `Error` (family, count), `Restore`, and one `Summary` per scope (peak instances, drops, errors, worst batch time).

#### Tests

Reuse Ticket 22's infrastructure, with small focused fakes and pure functions and no dependency-injection framework. Read [`docs/testing/studio-mcp-quirks.md`](testing/studio-mcp-quirks.md) first.

1. **Specs through `RunAll`:** pure `CollapseVfxRules` (stage derivation, tiers, pulse phase and rate, budget distribution and the essentials floor, drop order, context, configuration validation) and controller lifecycle specs on small fakes. Whether specs can run client-side is a **check**.
2. **S1 solo Play,** forced Full then forced Lite, on the TEMPORARY accelerated schedule (**test-only**, never committed), with the Edit-mode integrity check afterwards.
3. **S2 two clients and six rigs:** a Participant and a Hub or Spectator client, late joins, spectate switches and a six-roster Layer 1 removal.
4. **The capture matrix:** each stage, light and dark avatars, overlapping Abilities, a Warden Swing and Slam over a Final-tier Warning, HUD damage over a Warning, a six-player Dais capture, and Lite with all post-processing off, followed by colour-vision simulation on the captures.
5. **A real low-end phone pass** and a controller pass, recorded by hand.
6. **Regression:** the Ticket 21 structural validator, the Ticket 22 specs, and its matrix rows for lifecycle, presentation state, six-roster and spectator.

Timing-critical cases are armed against an absolute server timestamp, as in Ticket 22. A Play test that consumed Sections is followed by an Edit-mode integrity check.

#### Completion criteria

**Stage 1 (all required):**

1. Stage derivation is correct for 0 to 4 consumed layers with Layer 4 enabled and disabled, including `CollapseLayer = 0` while Layer 4 is still present, and a late join computes the target with no replay.
2. The Unravel plays once per `(matchId, SectionId)`, never on restore, never for a Section the client never saw, and never before removal.
3. No instability effect exists for a non-Warning Section, before the first Warning or in any stable gap.
4. Pulse phase is identical for a given `removesAt`, peaks land on whole seconds within a frame, tiers switch at 10 s and 5 s, and no pulse exceeds 3 Hz in any tier.
5. Every effect part is `CanCollide`, `CanQuery` and `CanTouch` false and outside the authored Section folders. The Ticket 21 and 22 specs, the Dais isolation check and removal timing are unchanged, and Ticket 22's criterion 9 and matrix rows still pass.
6. Lifecycle: mid-Warning cancel, mid-Unravel end, a static Results hold, an interrupted Results still restoring, the Hub restore, and a 3-Match leak check (instances, connections, tweens, sounds and pools back to baseline).
7. Isolation: a Hub client's lighting and audio are unaffected while another client is a Participant, Participant to Spectator keeps the scope, and a lighting conflict is never overwritten by restore.
8. Forced Lite shows every essential within the caps, the tier is fixed during a Match, and unknown signals give Lite.
9. An injected error disables only its family after three, a fatal error restores Lighting, and a missing asset warns once while visual equivalents remain.
10. Configuration validation rejects any pulse rate above 3 Hz.
11. **Readability protocol:** for each capture a reviewer answers yes or no to "can I identify every Ability effect, the HUD and every lethal edge". Any "no" blocks.
12. The implementation checks are performed and recorded, and the fracture-floor versus Lite-budget check is resolved.

**Stage 2:** the effect caps and cleanup still hold including horizon, dust and hums (Lite has none), hums contain no ticks, the ambience bed stays under the combat ceiling, the criterion 11 protocol is rerun on the full capture matrix, and the identity checklist passes with the developer's sign-off.

**Stage 3:** the Last Stand is dormant with Layer 4 disabled and correct with it enabled (tested under a TEMPORARY configuration, with the shipped configuration restored), the Dais is readable with the light disabled, the six-player Dais capture passes criterion 11, and the transition, Results hold and cleanup are correct.

Each stage also needs a regression run and an Edit-mode integrity check, and **stops for the developer's approval**.

#### Implementation checks

- The holding-folder ancestry signal, and whether a large removal splits across packets.
- Client access to each Section's authored `CollapseLayer`.
- Where Ticket 22's client telegraph and audio modules live, since the ticket doesn't name them.
- Whether Ticket 21's interior boundaries have the authored lip treatment.
- The graphics-quality APIs, defaulting to Lite when unknown.
- Every other writer of the Lighting properties and the existing sound groups.
- The fracture representation against real Section counts and Ticket 21's part-count baseline.
- The pale-void look when looking down, per lighting technology.
- The HUD backing plates, and participant detection from `MatchState`.
- Hub to arena distance and positional audio bleed.
- That effect parts are excluded from client raycasts, including cosmetic projectiles, aim and camera.
- Ticket 22's "standing on Section" lookup, reused for Lite's beacon priority.
- The exact Ability and HUD colours, and the `VfxGroup` assignment for rim links and connectors.
- Whether specs can run in a client context.

#### Release gates (not implementation blockers)

- A **real low-end mobile Lite pass**, performed and recorded. Lack of immediate access to a device does not block implementation or Studio validation, and it is recorded as an outstanding gate. A failure matters only if essentials are missing or cleanup fails.
- **Final visual identity** requires the developer's approval, through a written identity checklist (erasure rather than fire or darkness, Riftlight only in its four places, a pale void, stone contrast kept, visible escalation between stages, a distinct Last Stand) and a stage-by-stage capture set.
- **The Tidal gate** from Ticket 22 decides whether Layer 4, and so Stage 3, ships enabled.
- Approval for any asset upload.

#### Evidence only (recorded, never gating)

Frame-time and instance-cap tuning, the luminance, contrast, haze and hue values, audio balance and identity, placeholder sound quality, whether the Unravel reads from a distance, fragment counts, and whether the whole thing feels like the Unmaking. A tuning note never keeps a stage open, except where performance makes the game genuinely unplayable or warnings unreliable.

#### Deferred

- Concurrent Matches and per-session Collapse state.
- A Hub restyle and arrival polish.
- The final sound set and music, and texture, mesh or skybox assets.
- The camera fallback.
- Warden, Monster and Milestone visual reconciliation beyond the colour roles, and the Warden's despawn visuals (Ticket 23).
- A permanent exposed-edge trim.
- An Unstable stage and Dais overtime (Ticket 22's).

#### Dependencies, risks and ADRs

- Depends on Tickets 21 and 22 being built, with the presentation metadata authored.
- **Risks:**
  - The scope is large, which is why it is staged. Stage 1 is a complete minimum.
  - Replacing Ticket 22's minimal presentation could regress its criterion 9. The regression run covers it.
  - Lite shows fewer beacons than Full. This is accepted: every Section keeps fractures and cut-lines, and the HUD names the layer.
  - A match that ends mid-Unravel hard-cuts the animation. This is rare and accepted.
  - A light avatar against the pale void is the worst readability case, which is why the outline fallback exists.
  - Tickets 23 to 25 may add colours and sounds before this is built, so the colour roles and the audio ceiling are rechecked against what exists then.
- **No ADR required.** The choices are cosmetic and reversible, derived from state Ticket 22 already replicates, and they change no authority, movement, hitbox, workflow or one-life decision.
- **Glossary:** `CONTEXT.md` gains **Unmaking**, **Riftlight**, **Unravel** and **Last Anchor**.

#### Review notes (2026-11-10)

An adversarial pass against Tickets 21 to 25 found **no blocking contradiction and no gameplay change**. What the review found and fixed in the documents:

- **Ticket 21's "and nothing more":** the Collapse non-goal now carries a dated note allowing presentation anchors, with no gameplay scope.
- **Beacons per layer:** "one `BeaconAnchor` per group" was wrong, because a Shard spans layers 1 to 3. It is one per `VfxGroup` per `CollapseLayer`.
- **Registry timing:** a one-shot registry build at the new `MatchId` could miss Sections that restore late, so registration is event-driven by Section appearance.
- **Results freeze:** Ticket 22 restores Sections when everyone is back in the Hub, so a participant or Hub spectator can still be mid-Results. Stage derivation freezes on leaving Running.
- **Highlight budget:** the conditional outline fallback must check the Highlight budget alongside Ticket 25's Highlight-style visuals.
- **Audio priority:** Ticket 25's cues and the Feedback sounds sit above Collapse audio.
- **None-context clients:** world effects and positional audio exist only inside a scope, consistent with Ticket 22's status-only Hub viewers.
- **Ticket 22's per-frame derivation:** compatible, because the pulse and growth derive from timestamps every frame while a Warning is active.
- **Readability criterion:** the "identifiable" test was subjective, so it is a yes or no reviewer protocol where any "no" blocks.
- **Stage 1 stage 4:** Stage 1 ships a plain preset, and Stage 3 adds the anchor, drain and ambience.
- **Unnamed modules:** Ticket 22's client modules are not named, so locating them is a **check** and no name is invented.
- **Dais isolation exemption:** client-side parts never exist in the authored map or validator, so the exemption is untouched.
- **Hum versus ticks:** hums are sustained tones, so Ticket 22's "no global per-second ticks" and the on-ground final-5 s cue stay intact.

Accepted, with evidence to follow: the recorded risks above. Correctly deferred: see Deferred. Review result: **no ADR required.**

### 27. Minimap and Objective Awareness

**Status: specified 2026-11-11 and reviewed adversarially the same day, not built, and staged.** Every design point was grilled with the developer and settled. The reasoning, rejected options and edge cases are in [`minimap-design.md`](minimap-design.md); this entry is the build contract. Values are **fixed** (an approved requirement, changed only by a recorded design change) or **tunable** (a starting value that evidence can change). **Stage 1 starts only after Ticket 22 is fully implemented, tested and accepted, including its two-client runs.** Stage 2 starts only after Ticket 23 provides the Warden's authoritative state. There is **no dependency on Ticket 24 or Ticket 26**.

#### Summary

A read-only 2D map of the Rift Crown, built from authored topology, in two sizes of the same art: the **Minimap**, a small persistent corner view, and the **Tactical map**, a larger non-modal expanded view. It answers three questions: where am I, which ground is left, and what is about to go. It is not a radar. It never shows another player, and it carries no countdown.

#### Stages and gates

- **Stage 1 (core).** The rendering spike, the authored layout, `MinimapRules`, both views, the Collapse states, the camera-facing marker, nearby dropped Kits, input and spectators. Needs Tickets 17, 17B, 21 and 22.
- **Stage 2 (Warden marker).** A small, independently testable addition. Needs Ticket 23. Stage 1 never waits for it.
- **One Studio driver.** Ticket 27 is not built in parallel with Ticket 22's remaining Studio work, and the Studio is driven by one agent at a time.

#### Scope (Stage 1)

- A baked 2D layout: one polygon per **Section** (15 in the Ticket 21 build) and one per **connector** (12). Not "six Shards": a Shard spans several layers, so the unit is the Section.
- `MinimapRules` (pure) and one client controller with one presentation model shared by both views.
- The Collapse states, the player marker, nearby dropped Kits, the Tactical map's labels, the input behaviour, spectator support and cleanup.
- Specs, a Studio matrix, and the `CONTEXT.md` and `code-structure.md` entries.

#### Non-goals (fixed)

- **Any other player, ever.** No enemy marker, last-known position, proximity indicator, combat ping or endgame reveal. Players find enemies by observation, sound, positioning and the Collapse.
- Rift Caches (**deferred**, see below), Riftlings and Sentries.
- Elevation of any kind: no ramps, ruin platforms, contours or height numbers.
- A countdown of any kind on either map. The Collapse HUD line and the local danger alert already carry the timing.
- Zoom, pan, waypoints, selectable markers, tooltips and manual filters.
- Touch, gamepad and `Esc` handling, and any on-screen button (the mouse is locked to the centre). These belong to a later platform-input ticket.
- A `ViewportFrame`, a live Workspace scan, a per-frame geometry rebuild, a map service or parallel map simulation.
- **No new RemoteEvent, no server code, no new replication, no new attribute.**
- Uploading any asset. Anything that needs one stops and asks the developer.

#### Settled design

| Topic | Decision |
|---|---|
| Views | One persistent Minimap and one Tactical map of the same geometry and orientation. The Tactical map is **non-modal**: movement, the camera and gameplay continue. |
| Orientation | **Fixed north-up**, from one tunable constant that defines north in world space. It is checked against the Rift Crown's azimuth convention (`ShardAzimuth`) when built. |
| Style | Cold stone-grey ground, a dark void, Riftlight rose-magenta for Warning only, a pale-cyan player marker. No live overhead rendering. |
| Player marker | **One** cyan triangle. Its position follows the active or spectated character's X/Z. Its orientation follows the **local camera's horizontal yaw**, so camera pitch never changes it. There is no character-facing marker, no second camera line and no angle threshold. |
| Topology | Safe connectors are solid, Risky are segmented, and Dash shortcuts are dotted. The boundary between ground and void is the Lethal edge. The Dais is its own Section polygon, **labelled "Dais" on the Tactical map only**. Shards are labelled with their `ShardId` on the Tactical map, and the player's own Shard is distinguishable. |
| Elevation | **None** (fixed). |
| Collapse states | **Intact**: normal ground. **Warning**: magenta fill, outline and a **static fracture pattern**, so colour is never the only channel. **Removed**: nothing on the Minimap, and a faint dashed outline on the Tactical map. Removal is atomic. The settle window is never drawn as safe ground. The Dais is never removed, and an omitted Layer 4 simply stays Intact. **Nothing is highlighted before its authoritative Warning**, and the map says nothing the Collapse HUD does not. |
| Timing | The map shows state only. It adds no countdown and no second timer. It reuses the existing authoritative Collapse presentation state, and **stalled or shifted schedules therefore need no map-side handling**. |
| Kit drops | A **neutral glyph** (independent of Ability and Rank) on both maps for a Pickup with `PickupFromElimination`, within **30 studs horizontal (X/Z) distance** (fixed) of the active or spectated character, re-filtered at about 5 Hz (tunable). It disappears with its Pickup, on collection or on a Collapse sweep. Cache and Warden drops carry no flag and never show. |
| Wardens (Stage 2) | Visible to everyone **from `MatchCrownOpensAt`**, on both maps, spectators included, at the fixed **`WardenHome`**, never at the live root. **One glyph** for Idle, Engaged and Returning: presence is the only signal. Removed when `WardenState` is Dead or its Section is consumed, and **hidden when availability is unknown**. A disappearing marker may indirectly say that a Warden fell. That is accepted objective information, not enemy tracking. |
| Priority | 1 the player, 2 topology and Collapse, 3 Warden, 4 Kit drops. **Deterministic, presentation-only overlap suppression**: a lower priority marker may hide under a higher one. Markers stay at their true projected position, with no clustering, displacement or animation. |
| Input | **Keyboard and mouse only.** **M** toggles the Tactical map. Pressing the primary attack (MouseButton1), an Ability slot (Q, E) or Dash (LeftShift) closes it, and the action is **performed as normal, exactly once**. F (the replacement modifier) and G (Spectate) do not close it. There is no `Esc` handling. It also closes on elimination and on any Match transition. |
| Spectators | The marker follows the `Spectating` target, the triangle follows the spectator's own camera, and Kit proximity is measured from the target. Switching target recomputes the map context. **No living target means no map.** Match end and Return to Hub clear it. A Hub spectator's own Hub character is never the map's subject. |
| Context | A map context exists for a Participant or a Spectator while the Match is Running (the Match's own phase attribute). With no context, the controller does no work. |
| Layout | The corner is chosen from the corners no HUD element uses today (top-left, bottom-left and bottom-right in the audited HUD), after a check in Play that the default Roblox chat and menu do not cover it. Sizes are tunable. Mobile-first readability is kept as portable constraints: no hover, large enough glyphs, high contrast, and nothing positioned against a fixed screen size. |
| Rates | The player marker and triangle update at about 20 Hz (tunable). Everything else is event-driven. Static layout is built once. Overlap suppression is recomputed only when state or layout changes. |

#### Architecture

- **`MinimapRules`** (pure, `Shared/Rules`): the projection from world X/Z to map space, the horizontal distance filter, overlap suppression and priority, the Collapse state derivation, and configuration validation. It never touches `Workspace`. The name avoids `MapRules`, which is Ticket 21's structural validator.
- **One client controller** draws both views from one presentation model. It is read-only and fully cleaned up, and one scope per Match and role owns every connection, loop and instance.
- **Collapse state** comes from Ticket 22's replicated attributes: the per-model `CollapseState` and the Workspace `CollapseWarning` and `CollapseLayer`. A consumed model is reparented into a **server-only** holding folder, so a client never reads a `Removed` value. **Removed means an authored id whose model is absent from the world while a Collapse is running.** State is derived and idempotent, so there are no revisions to reject. If Ticket 26's `matchId`-keyed accessor exists, the controller uses it, with no second Collapse model.
- **Baked layout.** One data module of polygons keyed by `SectionId` and by connector identity, generated once from the authored footprints by an Edit-mode tool and checked against the real map by an additive structural check. Where the check lives is recorded when built. The tool changes nothing in the map.
- **Kit drops** are read from the Pickups the client already receives. Nothing is added to what the server replicates.
- **Input.** The map observes the events `Input` already handles through a read-only hook. `Input` performs the gameplay action first and the map only closes itself.
- **Who builds what.** The main session builds `MinimapRules`, the spectator and lifecycle integration, and reviews everything. `studio-implementer` builds the layout bake and the cosmetic drawing from exact briefs. `studio-tester` runs the Studio matrix.

#### Implementation steps (Stage 1)

1. **The rendering spike, first and gated.** Choose how the polygons are drawn, with **no `ViewportFrame` and no upload**.
   - Verify platform support first: whether `EditableImage` is available in this experience and Studio. Do not assume it.
   - Candidates in order: `EditableImage`, then row-run Frames, then rotated convex-piece Frames.
   - Measure each on the real layout: instance count, memory, cost per state change, the cost of a static frame (which must be zero), and a proxy for low-end mobile (Studio's emulator and the MicroProfiler, **stated as a proxy**). Thresholds are set from the measured HUD baseline and recorded **before** choosing.
   - If only an uploaded image would work, **stop and ask the developer**.
2. **Verify identities.** Whether `FromSection`, `ToSection` and `ConnectorRole` identify each of the 12 connectors uniquely. If not, add an additive `ConnectorId` to the Ticket 21 metadata, as `CutLine` was added.
3. **The layout bake and its check, and `MinimapRules` with specs.**
4. **The Minimap and the Tactical map**, static, then the Collapse states.
5. **The triangle, Kit drops, the input behaviour and spectators.**
6. **Lifecycle, the leak review, measurements and the Studio matrix.**

#### Stage 2, the Warden marker

Reads `MatchCrownOpensAt`, `WardenHome` and `WardenState`, and adds the glyph at priority 3 and its removal rules. Stage 2 first verifies that a Warden's Section is readable on the client and, if not, hides the marker when the Warden instance is absent. It adds no replication.

#### Tests

- **`MinimapRulesSpec`:**
  - the projection, including the corners and the north constant
  - the horizontal filter at 29.9 and 30.1 studs, and a vertical offset of 5 never changing the result
  - the priority and overlap rules, deterministically
  - the Collapse state derivation for Intact, Warning and Removed
  - a missing model before the Collapse starts, which is not Removed
  - unknown state, which hides a marker
  - configuration validation
- **Structural check:** every Section and connector has exactly one polygon, ids are unique, and a deliberately broken copy (a missing polygon, a duplicate id) fails.
- **Studio matrix (by `studio-tester`, two clients, accelerated schedule as TEMPORARY test configuration and reverted):**

| # | Check | Expected |
|---|---|---|
| 1 | Topology | All 15 Sections and 12 connectors drawn, three connector styles distinguishable, edges match the real Lethal edges |
| 2 | Collapse | Warning appears with the server's Warning and not before. Removal is atomic. The Dais stays. An omitted Layer 4 stays Intact |
| 3 | Independent connectors | A connector removed on its own layer shows Removed while its Sections do not |
| 4 | Kit drops | Appears at 29.9 and not at 30.1 studs. Disappears on collection and on a Collapse sweep. A Cache or Warden drop never shows |
| 5 | Triangle | Pitch changes nothing. Yaw tracks the camera |
| 6 | Spectators | Target switching, elimination, a Hub spectator, Match end and Return to Hub. No living target means no map |
| 7 | Input | M toggles. Each combat input closes it and the action fires exactly once. F and G do not close it. It closes on elimination and Match end |
| 8 | Leakage | No enemy marker in any state, for a player or a spectator |
| 9 | Lifecycle | Five Matches in a row leave no duplicated connection, loop or retained marker |
| 10 | Performance | No per-frame rebuild, no work without a context, and the spike's measurements hold |
| 11 | Reference Kit | With `startingKit = "Reference"` no map exists and nothing runs |
| 12 | Stage 2 | The Warden marker is absent before `MatchCrownOpensAt`, fixed at `WardenHome` through a chase, and removed on Dead, on its Section's removal, and on unknown state |

#### Completion criteria

**Blocking:** matrix rows 1 to 9, 11 and 12 (12 for Stage 2); the rendering spike's decision recorded before any other step; no upload made; no new remote, server code or replication; the Reference Kit unchanged; the full spec run green. **Evidence only:** the low-end mobile proxy numbers, the corner placement, and whether players use the Tactical map at all, which are recorded for tuning and never keep the ticket open.

#### Implementation checks

- The `Spectating` target accessor: whether one is exposed, and a read-only one if not.
- Whether the arrival hold falls inside Running, and what the map shows then.
- The default Roblox chat and menu against the chosen corner.
- Whether `StreamingEnabled` is off, since the docs say so and it was not read live.
- The value of the Collapse's settle window, which this entry deliberately does not repeat.
- The Rift Crown's azimuth convention against the north constant.

#### Deferred

- **Rift Cache markers.** Reconsidered only if playtesting shows a discovery problem. Cache state is live information about other players, so any later version needs its own fairness review.
- **Ramp markers**, only if playtesting shows a genuine navigation problem.
- **Touch and gamepad input and safe areas**, in a later platform-input ticket. Until then the real-device mobile pass is a gate of that ticket and not of this one.

#### Dependencies, risks and ADRs

- Depends on Tickets 17, 17B, 21 and 22 for Stage 1, and Ticket 23 for Stage 2. Ticket 26 is an **optional** integration (see its amendment). Ticket 24 has no relationship.
- **Risks:**
  - The rendering technique is unproven, which is why the spike comes first.
  - The baked layout can drift from the authored map, which the structural check and a rebuild-on-change rule cover.
  - Ticket 22's presentation may change during its remaining runs, which is why Stage 1 waits for acceptance.
  - The fixed north-up map of six similar Shards can disorient, which the `ShardId` labels and the own-Shard mark address.
- **No ADR required.** The ticket is presentation over state that is already replicated and changes no authority, movement, hitbox or one-life decision. The rule that no map ever shows another player is recorded here and in the design archive, and a future ticket that adds scanning or pings must reopen it deliberately.
- **Glossary:** `CONTEXT.md` gains **Minimap** and **Tactical map**, and **Riftlight** gains the map's Warning fill.

### 28. Final Kill presentation: slow motion and the Win-cam

**Status: proposed 2026-11-11 by the developer, specified here as a staged idea, not settled, not built.** The open points below are the first thing to settle. **Nothing starts before Ticket 22 is accepted.** It changes presentation only: the Match's result, timing and authority stay the server's.

#### Summary

When a Match ends with a Winner, the last Elimination plays in **slow motion** on the clients, and the Results phase can open with a **Win-cam**: a replay of that last kill from a replay camera. Neither changes who won, when the server decided it or what the rules say.

#### What exists now (read 2026-11-11)

- `Tuning.match.endResolutionSeconds = 0.25`: when a Match first reaches one Alive player, committed attacks get this long to land. Not a phase.
- `Tuning.match.display.bannerSeconds = 3`: the end banner, before the results panel. `resultsSeconds = 10`: the Results phase.
- `bodySeconds = 3` and `voidBodySeconds = 1` are how long an Eliminated body stays. They are **not** a hold on the Match end.
- A Collapse Void death waits for the settle window (`collapseSettleSeconds`, 1.75 after Ticket 22's measurement) before the verdict.
- `Client.Spectating` exists. No replay, history buffer or slow-motion exists.

#### Scope (proposed)

- **Slow motion.** Client-side only, on the final Elimination of a Winner Match: the Cosmetic Projectile, effects, the victim's animation tracks and the camera's tween are slowed by a factor for a short real-time window, then eased back. The server's physics and timers never slow, and nothing the server decides waits for it.
- **Win-cam.** The server keeps a **bounded rolling history** of the Alive players' positions and facing, and of the killing hit's Cosmetic Projectile, for the last few seconds. At a Winner's `MatchEnd` it sends one clip. The clients replay it with cosmetic rigs and a replay camera, in the Results phase, then the normal results panel. The replay never reads live physics.
- **Cleanup.** Every rig, effect, camera binding and connection ends with the replay, the Match and the player's leave.

#### Non-goals (proposed)

- No change to Match rules, the end check, `endResolutionSeconds`, scoring or the result row.
- No slowing of gameplay while play is live, for anyone, ever. This is the end of the Match only.
- No replay of a Match with no Winner (`NoSurvivors`, Timeout), unless the open points say otherwise.
- No upload of any asset, and no sharing or saving of replays.

#### Open points to settle with the developer before any build

1. **Who sees it.** The victim, the Winner, the spectators and the Hub players: all of them, or only some? Does a victim who is already Eliminated and spectating see the slow motion?
2. **What counts as the last kill.** The Winner's last Elimination only, or also the last Elimination before the Winner's, a Void fall by a Collapse, or a push credit (Tidal or Hook)? A Collapse fall as the final event is the awkward one: nobody "killed".
3. **The cases with no killer.** `NoSurvivors`, Timeout, a leave, and a combat death 0.3 s before a removal that ends the Match as a Winner (seen in Ticket 22's S2 runs).
4. **Slow-motion length and factor.** A starting value to measure: for example 0.3 times speed for about 1.5 s of real time, and whether the end banner starts after it. It must stay inside `bannerSeconds` plus `resultsSeconds`, or those change deliberately.
5. **Replay length, camera and skip.** For example the last 3 s from a third-person orbit, with a skip input, and whether the panel waits for it. Does the Win-cam run before or inside the Results phase?
6. **History size and cost.** Players covered (all Alive in the last N seconds, or only the killer and the victim), sample rate, a hard cap per Match and the network size of one clip. All measurements, never guesses.
7. **Fidelity.** Rigs standing in for the characters, with their animations replayed or approximated, and what the replay shows for a projectile or a Cache.

#### Likely approach (to confirm after the points above)

- A pure `ReplayRules` with specs: a ring buffer with a size cap, trimming to a window, and the clip's shape. Raycasts and the clock are passed in, as the other `Rules` do.
- A server `Replay` recorder fed by the same tick the Match already runs, with no new authority and one new `Protocol` event for the clip, validated like every other payload.
- A client `WinCam` controller that builds the rigs from the clip, drives the replay camera and cleans up.
- **Rendering caveat:** testing runs at about 15 fps and the viewport has stopped rendering in earlier tickets, so slow motion and camera checks need real renders. They go in a Server & Clients session, with the developer eyeballing the result.

#### Tests (proposed)

- **Specs:** the buffer's cap, window, order and empty-clip cases, and the clip's validation.
- **Studio:** a 2-client Match to a Winner, the slow motion seen on each side, the replay shown from the Results phase, skipped and not skipped, and a Match with no Winner showing neither. Rows are written when the open points are settled.
- **Blocking:** the Match result and its timing are identical with the feature on and off, which the logs show.

#### Dependencies, risks and ADRs

- Depends on Ticket 22 being accepted (its `MatchEnd` timing, the Collapse settle window and its Eliminated lines). Related to Ticket 25 (Results) and `Client.Spectating`.
- **Risks:** a replay that disagrees with what really happened, a buffer that grows without a cap, a replay that blocks the next Match's start (the Hub window is about 15 s), and the cost of cosmetic rigs on a weak device.
- **ADR:** probably none, since nothing changes authority. A recorder that replicates other players' positions after the fact is a minor fairness point to confirm in the open points: the replay only shows what already happened, to players who are no longer playing.

## Later, after evidence from the run test

- non-Ability world items, under the principles above
- ~~the smallest Rift Collapse~~, now specified as [Ticket 22](#22-rift-collapse), with its presentation staged in [Ticket 26](#26-rift-collapse-visuals-and-atmosphere); what remains is what its evidence asks for (an Unstable stage, roster-band timing, overtime)
- ~~the first Rift Warden~~, now specified as [Ticket 23](#23-rift-warden), and ~~regular monsters~~, now specified as [Ticket 24](#24-regular-monsters); what remains is an elite Dais Warden or boss, Monster Families, Relics and Rank 3 behaviour changes (Match progression is now specified as Milestones in [Ticket 25](#25-match-milestones), not XP)
- the real Hub, with batches of players into several Matches at once
- testing two slots against three, once there are about 6–8 Abilities
