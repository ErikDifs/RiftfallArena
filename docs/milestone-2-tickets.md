# Milestone 2 tickets

The ordered work for the second milestone: the **run progression loop**, **explore and fight PvE → find Abilities → change the Kit → fight players with it**. It tests whether Riftfall gets more interesting when a player's Kit develops during a run, not whether more content can be added.

Being settled from 2026-11-08, before any building. Work stops after each ticket until the developer gives the go-ahead, as in [Milestone 1](milestone-1-tickets.md).

## Checkpoints

| Date | Checkpoint | Contents |
|---|---|---|
| 2026-11-08 | Published place `134091394433436`, version 51 (private) | Ticket 15: the per-player Kit, its attributes and HUD, the Equip Lock, `ChargingRank` and the Starting Kit setting, Reference by default |

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

**Not yet published.**

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
