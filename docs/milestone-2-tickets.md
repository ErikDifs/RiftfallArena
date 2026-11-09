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

## Later, after evidence from the run test

- non-Ability world items, under the principles above
- the smallest Rift Collapse, if timeouts are more than occasional
- monsters, Elites and Rank 3 behaviour changes
- the real Hub, with batches of players into several Matches at once
- testing two slots against three, once there are about 6–8 Abilities
