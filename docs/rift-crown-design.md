# Rift Crown: Ticket 21 design notes (working document)

Working notes for Future Ticket 21, the first real Arena layout. Nothing here is built, and nothing is final until a point is marked **Settled**. When every point is settled these notes become the Ticket 21 entry in the milestone's ticket file.

**Terms.** "Arena" is ambiguous: `Workspace.Arena` is the Reference Kit / Stage 3 environment and "Riftfall Arena" is the game. In these notes the new map is the **Rift Crown**, and its pieces are **fragments**. `Workspace.MatchMap` is the existing greybox it will replace. (Glossary entries for Rift Crown and Fragment get added to `CONTEXT.md` once Point 1 is settled.)

## Facts from the existing docs and code that constrain the design

- The current `MatchMap` is a 250 × 250 greybox with **6 spawn pads** at r ≈ 105, **10 outer Caches** (r ≈ 80), **3 central Caches** (r ≈ 26) and **3 Sentries** (r ≈ 20) in three walled bays. The loot bag is exactly 16 Ability drops (10 × 1 + 3 × 2), so the Cache counts matter. Ticket 18 says size and layout are tuning values, not protected.
- Spawns go through `MatchRules.assignSpawns`, which already deals up to 6 pads so 2–6 players never share one.
- **Rift Hook's pull is flattened to XZ**, moves the target to about 4 studs from the caster in 0.25 s, and stops at static geometry. It never passes through walls, but nothing stops it crossing a gap. Range 30 / 36. Tidal Bolt Knockback is about 25 studs. These two numbers decide how wide a "deadly" ledge or gap needs to be (Points 4 and 8).
- Sentry range is 45 studs with a line-of-sight check against the environment roots, and Sentry hits Knock back about 20–25 studs. Sentries next to a void edge would be cheap kills, so Sentry slots keep clear of edges (Point 7).
- Falls already count as a death in Milestone 2 ("Any death counts: to a player, a monster or a fall"), but no void exists yet in `MatchMap`.
- The Match is 480 s (`maxSeconds`), and a target of 4 players is the starting point, 2–6 still workable.

---

## Point 1. Fragment structure

### What the point has to decide

The number of major fragments, whether they are alike or different, what each is for, and whether the centre is one of them. Everything later (dimensions, connectors, spawns, Collapse) hangs off this.

### Consequences and failure modes to design against

- **Fragment = territory.** If each spawn owns a whole fragment, the opening is calm (good for Gather and Kit building) but the first fight happens at a connector, which makes connectors the only place PvP starts. Too few connectors means death funnels; too many means the fragments stop mattering.
- **Spawn count vs fragment count.** The code deals 6 pads. With 5 fragments one fragment must hold two pads, with 7 outer fragments one has none. A 6-outer layout maps 1:1.
- **Unused fragments with 2–3 players.** They must be worth visiting (Caches, a Warden slot) or they become dead space and the map feels too big.
- **Identical fragments feel generic.** Identical *roles* are fine for spawns, but shape, edge profile and what sits on them should differ, or this is a pie chart.
- **Collapse wants a clean order.** Outer ring → inner → centre needs the fragments to sit in rings, with the centre a single place that is last to go.
- **Solo-dev cost** is mostly fragment count × connector count. Each extra fragment adds ~2 connectors, Cache/Sentry markers and a pass of testing.

### Options

**A. Six Shards + the Crown (7 fragments).** Six outer shards in a ring, one spawn each, around a central ruin that is the 7th fragment (the Crown). Shards are roughly equal in role and area, but irregular in shape and edge profile (±20% size, different ledge shapes). Each shard links to both neighbours (the rim route) and to the Crown by one inward route, with the link types deliberately mixed (Point 4).
- Opening: everyone loots their own shard and the edge of the next, nobody is near anybody.
- 2–6 players: a 1:1 spawn map. Unused shards stay as normal loot-and-risk terrain, so a 2-player match means "4 spare shards to roam".
- Environmental kills: every shard has a full outer edge, plus the rim gaps between shards.
- PvE/Caches: outer Caches on shards (two per shard fits the existing 10–12), Sentries and Wardens on the Crown and the inward routes.
- Collapse: shards go first, Crown last. Radial and simple.
- Solo-dev complexity: **lowest.** 7 pieces, 6 near-repeatable.

**B. Five Shards + the Crown, deliberately uneven (6 fragments).** Two large "home" shards and three smaller, riskier shards. The large ones carry two spawn pads each (at opposite ends), the small ones carry none and exist as contested loot islands.
- Opening: 2–3 players each start on their own large shard, with 4–6 sharing, so spawns are closer together.
- 2–6 players: fits less cleanly, because two pads on one shard breaks "everyone starts alone on their own ground" at 4+ players.
- Environmental kills: better, since the small shards are all edge.
- PvE/Caches: strong contested loot on the small shards.
- Collapse: the small shards go first, which strands nobody only if the large shards stay linked.
- Solo-dev complexity: **medium**, with uneven geometry and per-count spawn rules.

**C. Three Anchors + Three Satellites + the Crown (7 fragments).** Three big fragments hold the spawns (two pads each, far apart), three small satellite fragments sit between them as risky connectors with loot, and the Crown is in the middle.
- Opening: the most designed-feeling map, with three clear "territories" and obvious high-risk islands between them.
- 2–6 players: 2–3 players start one per anchor. 4–6 means two players per anchor, 40–60 studs apart on the same piece of ground, which is the spawn-rush problem Point 6 wants to avoid.
- Environmental kills: very good, since the satellites are all ledge.
- Collapse: satellites first, then anchors, then the Crown. Good, but anchors are big, so collapsing them takes the longest to read.
- Solo-dev complexity: **highest**, as three fragment sizes, two spawn rules and special-case routes.

### Recommendation: **Option A, Six Shards + the Crown**

- It matches what the code and the loot bag already assume: 6 pads, 1:1 with shards, and `assignSpawns` stays unchanged.
- Every player count from 2 to 6 works without a per-count rule: the unused shards are normal terrain and are where the extra Caches sit.
- Collapse is the simplest possible: shards (outer), then the Crown (centre). An "inner ring" can come later by letting each shard collapse from its outer edge inward.
- It is the cheapest to build and iterate on. A shard is a repeatable unit, so I can build one well and vary five.
- The "generic ring map" risk is handled by giving the shards **different shapes and edge profiles** (one long and thin, one with a spur, one that is nearly round, and so on) and by making the connectors uneven in Point 4, not by changing the structure.
- The Crown counts as the 7th fragment but has a different job from the shards: it is the contested centre, the Warden/high-value location and the final-fight ground.

So the Rift Crown is **1 Crown + 6 Shards = 7 fragments**, which is the top of your 5–7 range.

### **Settled (developer approved Option A).**

- **1 Crown + 6 outer Shards**, 7 fragments in all, with **1 spawn per Shard**.
- Shards are **similar in gameplay role** but **irregular in shape, silhouette, edge profile and local geometry**. They reuse one construction language and are never literal copies.
- The Crown is the 7th fragment and the **contested centre**, not a spawn shard.
- Unused Shards stay fully playable and lootable at low player counts.
- The six-spawn assignment model (`MatchRules.assignSpawns`) stays intact, and there are **no player-count-specific map variants**.
- The topology stays broadly radial, so the future Rift Collapse can consume **Shards → inner routes → Crown**.
- **Standing constraint for later points:** each Shard should be independently viable as an opening area, but no Shard may become a self-contained safe island that lets a player avoid crossing into contested space for too long.

---

## Point 2. Overall dimensions

### Movement numbers the scale depends on

| Fact | Value | Source |
|---|---|---|
| Walk speed | 16 studs/s (12 while Charging) | `first-combat-prototype.md` |
| Dash | about 21 studs, 2 charges, about 4 s recharge per charge | Ticket 17 check A9, `first-combat-prototype.md` |
| Jump | about 8 studs horizontally at walk speed, **assumed from Roblox defaults and to be verified in `Config` and Studio** | assumption |
| Rift Hook | range 30 (Rank 1) / 36 (Rank 2), pulls the target to about 4 studs from the caster in 0.25 s, XZ only, stopped by geometry, **may pull past a ledge** | Ticket 19 |
| Tidal Bolt tap Knockback | about **5 studs** (25 studs/s for 0.2 s) | `studio-mcp-quirks.md` |
| Sentry | range 45, shots about 3 s apart | Ticket 18 |
| Current `MatchMap` | 250 × 250, pads at r ≈ 105, first Cache 2–3.5 s from a pad, opposite pads about 210 studs (13 s) apart | Ticket 18 |

### Consequences and failure modes

- **A ring costs area, not distance.** A Rift Crown of the same diameter as the old 250 × 250 square walks about the same but has roughly half the floor, because the gaps are void. Travel time is what keeps Ticket 18 and 20's pacing numbers valid, so I scale on travel time, not on floor area.
- **Too small** and the six spawns are inside each other's Sentry/Hook range (36), the opening collapses into immediate fights, Shards can't hold 2 Caches plus a Sentry or Warden slot without clutter, and there is nothing to Collapse.
- **Too large** and 2-player Matches wait 20+ s to meet, Shards look empty with only 10 outer Caches, and the Match timeout (480 s) depends on a Collapse that doesn't exist yet. Ticket 18 deliberately has no Collapse, so the map has to converge on its own.
- **Dash changes gaps more than time.** One Dash saves about 1.1 s of walking, but players keep Dashes for fights, so I plan on walking. What Dash does change is which gaps are crossable (see the bands below).
- **Hook crosses every intended gap, in one direction.** It never moves the *caster*, so it isn't a grappling traversal tool. It pulls a *target* over a gap onto the caster's side. The kill case is a caster who stands within about 4 studs of an edge with void ahead: the 4-stud landing point is over the void. That is a deliberate environmental kill that costs the caster their edge position. Whether gravity acts during the 0.25 s pull (a target pulled over a wide gap might drop short) isn't known from the docs, and **has to be measured when the map is built**.
- **Tidal Bolt only kills within about 5 studs of an edge** (more with repeated hits). The deadly lip is therefore a narrow band, not "the whole perimeter", which supports "walking around shouldn't feel dangerous".
- **Sentry range of 45 covers a Ø60 Crown from anywhere on it.** The existing three-bay centre sits within r ≈ 26, so it fits a Crown of about Ø60–64 unchanged.

### Gap bands (apply to every option, in studs)

| Band | Width | Crossed by |
|---|---|---|
| Jumpable | up to about 6 | a jump, from a standing start |
| Dash gap | about 10–16 | Dash, or Dash plus jump (to be verified) |
| Bridge gap | about 18–24 | only a bridge or ramp. A Rank 1 Hook can still span the widest of these edge to edge |
| Never | over about 30 | Hook can no longer cross, and "gaps" stop reading as one structure |

### Options

All figures are starting values. Walk times assume 16 studs/s, with "routed" adding about 20% for a bridge detour.

**1. Tight: Ø about 200** — Crown Ø 44, inner gap 14, Shard depth about 60, Shard width about 60, area about 3,000, rim gap 8–16. Spawn r ≈ 85.

**2. Medium: Ø about 260** — Crown Ø 64, inner gap 16, Shard depth about 82, Shard width about 80, area about 5,000, rim gap 10–24. Spawn r ≈ 112.

**3. Large: Ø about 340** — Crown Ø 80, inner gap 20, Shard depth about 110, Shard width about 105, area about 9,000, rim gap 12–28. Spawn r ≈ 150.

| Walk time | 1. Tight | 2. Medium | 3. Large |
|---|---|---|---|
| Across one Shard | 3.9 s | 5.1 s | 6.9 s |
| Spawn → first connector | about 2 s | about 2.5 s | about 3.5 s |
| Between neighbouring Shards (spawn to spawn, routed) | about 6.5 s | about 8.5 s | about 11.5 s |
| Spawn → Crown (routed) | about 4.7 s | about 6 s | about 8.3 s |
| Opposite spawns (2 players, routed) | about 12 s | about 17 s | about 23 s |
| Across the whole Arena | about 12.5 s | about 16 s | about 21 s |
| Total floor | about 20,000 | about 33,000 | about 62,000 |
| Floor per player at 4 players | about 5,000 | about 8,000 | about 15,500 |

**How each holds up**
- **2 players (opposite pads, 3 Shards apart):** Tight meets in 12 s, which is a spawn rush. Medium meets in about 17 s, close to the old map's 13 s once routing is counted, and leaves time for a first Cache. Large waits about 23 s, which is dead time with 4 empty Shards.
- **6 players:** Tight puts neighbouring spawns 85 studs apart, inside two Hooks. Medium puts them 112 studs apart, beyond Sentry and Hook range but only about 8 s walking. Large is comfortable but empty.
- **Loot (10 outer Caches, 3 central, 16 Ability drops):** about 1.7 outer Caches per Shard. Tight is cramped once a Sentry or Warden slot is added. Medium gives about 2 Caches per 5,000 studs (spacing near 50), with room left for Sentry/Warden slots. Large leaves Shards looking empty unless the loot count grows.
- **Pacing carry-over:** Medium keeps spawn → first connector at 2–3.5 s, the same range Ticket 18 set for spawn → first Cache, so Ticket 20's pacing evidence still applies.

### Recommendation: **Option 2, Medium (Ø about 260)**

- Crown **Ø about 64**: it holds the existing three-bay centre (Caches r ≈ 26, Sentries r ≈ 20) without redesign.
- Shards **about 80 × 82**, irregular, area **4,000–6,000** (they may differ by about ±20%).
- Inner gap (Shard to Crown) **about 16** and rim gap (Shard to Shard) **10–24**, using the gap bands above. Gap widths are the one thing I'd tune first after the first greybox walk-through.
- **Fixed size.** It doesn't scale with player count, as locked in Point 1.
- Build the greybox at these numbers and play it before any detail pass. Ticket 18 already says size and layout are tuning values, so resizing at the greybox stage is the intended way to correct this.

### **Settled (developer approved Option 2, Medium).**

- Rift Crown diameter **about 260**, Crown **about 64**, typical Shard **about 80 × 82**, individual Shards **±20% in usable area**, inner Crown gap **about 16**, rim gaps generally **10–24**.
- **Fixed size for 2–6 players.** The physical Arena never scales with player count.
- Target walking times: spawn → first connector **2–3.5 s**, spawn → Crown **about 6 s**, neighbouring spawn regions **8–9 s**, opposite spawn regions **about 17 s**, full traversal **about 16 s**.
- Each Shard keeps enough floor for loot, future Warden/Sentry slots and normal combat without feeling sparse.
- **The gap categories are approved as a concept. The bands are provisional**, and the exact jumpable distance is not locked because the jump assumption is unverified:
  - **Normal traversal gap:** needs no special movement. The maximum comes from measured Roblox jump behaviour.
  - **Dash shortcut:** about 10–16, traversable with Dash but not ordinary movement.
  - **Bridge-required gap:** about 18–24.
  - **Avoid** deliberate Shard-to-Shard gaps above about 30.
  - **Rift Hook range is never proof that a route is traversable**: Hook moves the target, not the caster.
- **Greybox validation must measure:** reliable running jump distance, standing versus running jump behaviour, Dash crossing distance, whether gravity affects targets during Rift Hook's 0.25 s pull, and whether Hook reliably carries targets across the intended gaps. **If the measurements contradict the bands, adjust the bands, not the Arena scale.**
- **Standing constraint:** a Dash shortcut is a meaningful positional choice, not a mandatory route. Every major region must stay reachable without spending Dash.

---

## Point 3. The Crown

### Correction to my Point 2 claim

I said a Ø64 Crown "holds the existing three-bay centre unchanged". That is wrong now that the edge is void. The old centre had Caches at r ≈ 26 and Sentries at r ≈ 20, inside walls. On a Ø64 Crown (r = 32) a Cache at r 26 is **6 studs from the lip**. A Sentry at r 20 pushes players *outward*, toward that lip, with each Tidal Bolt-style hit (about 5 studs, and a central Cache takes about 8 s to break, so 2–3 Sentry shots). That is exactly the "cheap unavoidable ledge death" the brief wants to avoid. The old numbers don't transfer, and the layout below moves them.

### Facts that shape the Crown

- A central Cache has 120 health (about 8 s of Basic Attack), drops **two** Abilities about 10 studs apart, and is guarded by a Sentry. Sentries have range 45 with a line-of-sight check, so one Sentry covers the entire Ø64 Crown unless something blocks the line.
- Ticket 18 used radial walls so that three Sentries don't all acquire one player. Any Crown has to keep a version of that.
- **A Sentry's push goes away from the Sentry.** Where the Sentry stands decides which way players get pushed.
- Usable footprint: Ø64 is about 3,200 studs². Excluding the outer 5-stud lethal band it is about 2,300. A comfortable fighting core (r ≤ 22) is about 1,500.

### The opening-rush problem (this is real, not only a layout question)

A central Cache gives **two Abilities from one break** and takes about 8 s, while an outer Cache gives one in about 5 s. With the spawn-to-Crown time at about 6 s, a player who sprints to the Crown can have **a full Kit at about 14 s, against about 25 s for the home route**. The cost is a Sentry plus up to five rivals arriving at the same moment. For an aggressive player that can be the best opening, which is the outcome you want to avoid.

The Crown's geometry can only reduce this. It can't remove it. What it can do: make the first minute at the Crown crowded and exposed (screened bays, Sentries, nothing safe to stand on while breaking), and not offer a safe approach. The remaining levers are values and timing: central Cache health, Sentry fire rate, the central loot, or a delay before central Caches can be damaged. Those belong to **Point 7**, and I'll bring it back there with options instead of deciding it here. I'd also expect the first playtest to be the real test.

### Options

**A. Walled Bays (the Ticket 18 centre, adapted).** Flat disc Ø64, a solid core r ≈ 10 and three radial walls (about 20 high) making three bays. Each bay has a Cache and a Sentry.
- Usable footprint: about 2,900 (core excluded). Single level.
- Entrances: each bay is open on the rim side, three wedges of 120°, no doors.
- Around without entering: only along the rim, which is the lip.
- Sight lines: very short inside bays, so Sentries stay independent. Hook and Tidal Bolt lanes are cut by walls, and Hook is stopped by the walls.
- Env. kills: the lip is deadly all round, and the bays are dead ends, so a player cornered in a bay has nowhere to go but the lip.
- Fit: Stone Guard good, Arc Burst and Tidal Bolt in a bay good, Hook poor (walls cancel pulls).
- Warden: the core is dead space. Nothing can sit centrally and be seen.
- Final fight: bays isolate fighters, so 2-player finals are cat-and-mouse.
- Attraction: the walls make the Crown look like a fortress from outside, which is a strong pull.
- Cost: lowest, because most of it exists already.

**B. Terraced Crown with Ribs (recommended).** Disc Ø64 with a **raised central dais** and three short **ribs** (partial curved walls) on the terrace instead of a core and long walls.
- Shape: a lower terrace (r 14–32, y = 0) and a flat dais (r < 14, raised about 5), joined by three broad ramps at 60°, 180° and 300°. The dais face is vertical between ramps, but a 5-stud step is probably jumpable (to be verified), so the ramps are never chokepoints.
- Ribs: three curved wall segments about 14 long and 8 high, at 60°, 180° and 300° on the terrace at r ≈ 24, just outside each ramp foot, so they screen the ramps and the arrivals around them. They break Sentry sight lines between bays without closing anything. The gaps between ribs are about 25 wide.
- Usable footprint: about 3,100 (terrace about 2,500 plus dais about 600). Two levels.
- Entrances: **none as doors.** The whole rim is open, and every bridge landing arrives on the terrace.
- Around without entering: yes, via the rim bridges (Point 4), or along the terrace edge, which is the risky edge-walk.
- Sight lines: the dais is a high point for Arc Burst and Tidal Bolt over the terrace. Ribs cut cross-Crown lines without isolating anyone.
- Env. kills: the lip is dangerous for **about 5–6 studs only.** Nothing valuable sits within 10 studs of it (see placement below). The dais face drops about 5 studs and is **not** lethal, so the Crown's centre is the safe-ish part and the rim is the dangerous part.
- Fit: Stone Guard good, with the dais face as a retreat. Arc Burst and Tidal Bolt good. Hook works across the open terrace, and the sweep correctly stops against the dais face, so pulling a target up onto the dais isn't possible (Point 5 decides whether that is acceptable).
- Warden/high-value: the dais is a natural 28-stud arena for one future Warden or high-value objective. Its knockback pushes onto the terrace, not toward the void (the dais edge is 18 studs from the lip).
- Final fight: a flat terrace ring around a defensible dais reads well for 2 players (they use the dais and ribs) and for 5–6 (multiple lanes).
- Attraction: the dais shows from a distance, but it is only about 5 high, so it isn't a beacon.
- Cost: medium. One disc, one cylinder, three ramps, three ribs, three Sentry pedestals.

**C. Open Plaza with Standing Stones.** Flat disc Ø64, no core, no dais, five or six free-standing stone pillars as cover.
- Usable footprint: about 3,100, single level.
- Entrances: fully open. No chokepoints at all.
- Sight lines: open, so **all three Sentries acquire the same player** and a player in the middle takes triple fire. That undoes Ticket 18's separation unless the Sentries are removed from the Crown.
- Env. kills: the lip is all there is.
- Fit: Hook, Tidal Bolt and Arc Burst all strong. Stone Guard weaker (little to retreat to).
- Warden: central, easy to read. Final fight: the most open and the most kite-able. At 2 players it can become a chase around the stones.
- Attraction: moderate. Cost: lowest of the three, but it needs a different Sentry arrangement.

**Rejected variant: a Hollow Crown** (a ring around a central void well). It makes the centre, where the Warden and final fights happen, the most lethal place on the map. That conflicts with "most fighting ground should be broad and safe".

### Placement rules for B (they also apply to any option)

- **The lip rule:** no Cache, drop point or Sentry-covered standing spot within about **10 studs of a lip**. The old r ≈ 26 Caches are moved inward.
- **Sentry push rule:** a Sentry's push line must not point at a lip within about 10 studs of the player it hits. Practically, the Sentry stands on the void side of its Cache, so its push goes toward the dais, or the line is blocked by the dais face. A Sentry that pushes outward from the centre is not allowed.
- **Bays:** three, at 0°, 120° and 240°, the azimuths **between the ribs and ramps**. Each has a Cache at about r 19 (just outside the dais foot, 13 from the lip) and a Sentry at about r 27 on a low pedestal behind it facing in. Drops land ±5 studs across the bay, which keeps them at about r 19 as well.
- **Ramps and landings:** the dais ramps sit at the rib azimuths, never at the Sentries. Bridge landings (Point 4) should arrive on the rib sides, so arriving players are screened from the nearest Sentry and no landing sits within about 15° of a Sentry azimuth.
- **Dais:** empty at the start, the future Warden slot. It holds no Cache in v1.

### Recommendation: **Option B, Terraced Crown with Ribs**

- It keeps the Ticket 18 idea (three independent Cache-and-Sentry bays) without walls, doors or dead ends.
- It fixes the lip problem the old centre would have had.
- It gives the Crown a real shape: the dais is a visible, defensible centre for the Warden and the final fight.
- It stays a few primitives, so it is buildable and re-tunable by one person. Heights stay as small numbers (about 5 and 8) so Point 5 can still change them.

### **Settled (developer approved Option B, Terraced Crown with Ribs).**

- **Crown about Ø64.** A central **dais** of radius about 14, raised about 5, reached by **three ramps**, **empty in Arena v1** and reserved for a future elite Rift Warden or high-value objective. The terrace stays open enough for normal combat.
- **Three curved ribs** at roughly **60°, 180° and 300°** give cover and break sight lines without enclosed bays or dead ends. **Cache/Sentry bays** sit at roughly **0°, 120° and 240°**.
- **Placement rules (locked):**
  - No Cache, loot drop point or intended combat-standing spot covered by a Sentry within roughly **10 studs of a lethal lip**.
  - Crown Sentries are positioned so their displacement pushes players **inward**, never toward the void. A Sentry that creates routine outward knockback toward a lethal edge is invalid.
  - Central Caches at about **r ≈ 19**, central Sentries at about **r ≈ 27** on low pedestals, Ability drops roughly **±5 studs across the bay**, away from the lip.
  - Connector arrivals approach from the **rib sides**, never directly into a Sentry lane, and no bridge landing within roughly **15°** of a Sentry firing lane.
  - The dais face is a non-lethal ~5-stud transition. The Crown's centre is generally safer than its outer rim, and **the outer rim is the primary environmental-kill zone**.
- **Principle:** *The Crown should reward controlling space, not reward hiding behind walls or forcing players through narrow doorways.*
- **Rejected:** the old walled-bay layout as the final Crown (it suppresses Rift Hook play and creates dead-end combat), and the hollow-centre concept for Arena v1 (the centre must not be the most lethal zone).
- **Deferred to Point 7, recorded explicitly:** *With current central Cache behaviour, sprinting directly to the Crown may produce a full Kit substantially faster than following the intended home-Shard progression path. Crown geometry alone should not solve this. Point 7 must decide how central loot/PvE value and availability prevent centre-rushing from becoming the dominant opening strategy.* Cache health, Sentry fire rate, loot quantity and activation timing are **not changed** in this ticket's design until then.

---

## Point 4. Connectors and routes

### Vocabulary

- **Spoke:** a connector between a Shard and the Crown. **Rim link:** a connector between two neighbouring Shards.
- **Gate:** the two Crown landings that flank one rib.

### Fixed geometry (from Points 1–3)

Shard centres sit at **30°, 90°, 150°, 210°, 270° and 330°**, so each Shard is exactly between a bay (0/120/240) and a rib (60/180/300). That makes the natural landing for each Shard's spoke one **20° on the rib side of the Shard**: **40°, 80°, 160°, 200°, 280° and 320°**. Each landing is at least 40° from every Sentry lane (the rule needs 15°), and the six landings group into **three Gates** of two landings around each rib. The Shards of one Gate are a **pair**: (30, 90), (150, 210), (270, 330). Spokes are about **16 studs long**, since that is the inner gap.

### What displacement does on a bridge (this decides the widths)

- **A Hook only kills when its 4-stud landing point is over void.** The target ends 4 studs from the caster on the line from target to caster. A caster standing 4 or more studs inside solid ground can't kill with a pull. So a **Hook kill needs a caster at, or within about 4 studs of, a lip, with void between them and the target.** The caster is exposed too (Tidal Bolt, Sentry hits, other players), which makes it a fair trade.
- **A Hook pull that runs along a bridge's own axis is safe**, because the landing is on the bridge. A pull that runs *across* the bridge, from a caster on a flanking lip, ends over void. Anyone crossing a bridge within about 30 studs of a flanking lip can be pulled sideways. **A bridge with a solid parapet stops that**, because the pull stops at static geometry (the pull sweep is a character-sized spherecast, so the parapet must be roughly torso height, to be verified).
- **A Tidal Bolt tap moves 5 studs.** On a bridge 6 wide the centre is 3 studs from each edge, so **one sideways tap kills.** At 10 wide, one tap puts you on the edge and a second kills. So bridges are at least 8–10 wide, to keep the lethality at two hits, not one.
- **Spawn pads need 20 or more studs of margin from any lip** (Point 6 confirms).

### Route taxonomy (proposed)

| Type | Structure | Dimensions | Displacement |
|---|---|---|---|
| **Safe connector** | A broad bridge or causeway with parapets, needs no Dash | width 14–18, length up to about 24 | Parapets stop sideways Hook pulls and Tidal pushes. Death needs a deliberate effort, e.g. fighting at the open end. |
| **Risky connector** | A narrow bridge with no parapets, needs no Dash, visibly cracked or broken | width 8–10, same lengths | Meaningful Tidal edge pressure (two hits), and sideways Hook pulls from flanking lips can kill. This is where the map's displacement play lives. |
| **Dash shortcut** | A bridgeless gap across a **pinch** between two Shards, with a solid ledge on each side and a 6-stud apron before each lip | gap about 10–14, narrower than the 21-stud Dash | Failing a Dash is a fall. Gap crossers are exposed to Hook from flanking lips. It's never the only route. |
| **Jumpable gap** | Not in the connection graph. Allowed later only as a local crack inside a Shard, once the jump distance is measured | measured | Not a connector. |

A Dash shortcut is always an **additional** route beside a bridge route to the same region, and every region stays reachable without Dash. Safe connectors are a **choice of route, not a separate map**: the Safe and Risky bridges of one Shard land at the same Gate.

### Options for the connection graph

**A. Full Wheel (12 bridges).** All six spokes and all six rim links exist, all the same type.
- Per Shard: 3 exits. Direct Shard → Crown for all. Neighbours all connect. Rotate around without the Crown: yes, completely.
- Crown chokepoint: no. Redundancy: very high, and a Shard can't be trapped. Disengage: easy. Spawn rush: every neighbour is 8–9 s away, so it is the most rushable.
- Displacement: every rim gap has a bridge, so most displacement happens on identical bridges. Cheap Hook kills are avoidable but everywhere.
- Collapse: clean. Readability: high, because it's uniform. It is also the **generic ring map** Point 1 wanted to avoid. Cost: 12 bridges, but one prefab type.

**B. Gated Pairs, typed (recommended): 10 bridges plus 2 Dash gaps.**
- **Spokes: all six.** Within each Gate pair, one is Safe and one Risky: Shards 30, 150, 270 get **Safe** spokes, and 90, 210, 330 get **Risky** spokes.
- **Pair links (3, Safe):** 30↔90, 150↔210, 270↔330, across gaps about 18–20 (bridge-required) at the Gates' sides.
- **Inter-pair links (3):** 90↔150, 210↔270, 330↔30 across the bay sides. **One is a Risky bridge** (210↔270), and the **other two are Dash shortcuts** (gaps about 10–14), so the three aren't the same.
- Per Shard: 2–3 bridges (30, 90, 150, 330 have 2 bridges and one Dash gap, 210 and 270 have 3 bridges).
- Direct Shard → Crown: yes for all six, one landing each, none shared, 20° off a rib, 40° off a Sentry lane.
- Rotate around without the Crown: only partly (150–210–270–330 is one non-Dash chain, and the 30/90 pair joins by Dash gap or the Crown). Going around the Crown's perimeter on the terrace (without the dais) is possible and passes a bay, which is Sentry exposure.
- Crown chokepoint: no, six landings and a terrace ring. Redundancy: every Shard has a Safe path to the Crown within one hop. Trapped on a Shard: no.
- Disengage: a Dash shortcut disengages fast but costs a Dash. Spawn rush: pair partners are the quickest to meet (about 8.5 s), inter-pair neighbours about 8 s with a Dash and about 12 s without.
- Displacement: Risky spokes and the Risky rim bridge, plus the Dash gaps, are the kill places. Safe bridges are not.
- Collapse: clean (see below). Readability: three visibly different connector types. Cost: medium. Three prefabs and 12 placements.

**C. Three Gates plus a full ring (9 bridges).** Only Shards 30, 150, 270 have spokes. All six rim links exist.
- The other three Shards reach the Crown through a neighbour, so spawn → Crown is about 9 s for them and about 6 s for the others. **That breaks the locked 6 s target and makes three spawns structurally better.** Reject.

**Rejected outright: spokes only** (no rim links) makes the Crown the single route between any two Shards, which is an unavoidable chokepoint.

### Where B misses a locked Point 2 target (flagging it)

Point 2 targets **8–9 s between neighbouring spawn regions**. In B, the two inter-pair links that are Dash shortcuts give about 8 s with a Dash but about 12–13 s on foot (through the Crown terrace). The third inter-pair link and all three pair links meet the target. I think the 12 s on-foot case is the right trade because it is the reason a Dash shortcut means something, but it is a deliberate deviation, and I'd rather you decide than discover it. If you'd prefer to keep 8–9 s everywhere, make all three inter-pair links Risky bridges, which gives A's 12 bridges with a Risky/Safe mix and no Dash shortcuts.

### Displacement safety review for B

| Geometry | Verdict |
|---|---|
| Caster at a lip across a rim gap, target visibly across the gap | **Fair Hook kill.** The caster is exposed, the target can dodge or break line. This is the intended play. |
| Target on a Risky bridge, caster on a flanking lip | **Fair**, because it is a chosen risk. The crossing is visible, and it is a Risky bridge by definition. |
| Target on a Safe bridge | **Not killable** by a sideways pull (parapet), and the axial pull lands on the bridge. |
| Caster standing mid-bridge on a Risky bridge | Fair, and the caster is the more exposed one. |
| **Cheap:** a landing where the only approach passes a lip within about 8 studs, with no parapet | **Avoid.** Landings have a 6-stud apron and lips at least 8 studs from the approach line. |
| **Cheap:** spawn pads, Cache or drop points within about 10 studs of a lip | **Avoid** (locked for the Crown, extended to Shards in Points 6–7). |
| **Cheap:** a Risky bridge narrower than 8 | **Avoid.** One Tidal tap would kill. |
| Meaningful Tidal pressure | Risky bridges, Dash-gap aprons, and the outer 6 studs of every Shard. |
| Accidental falls | Dash toward a lip or off an unfenced edge. Mitigated by 6-stud aprons, no loot within 10 studs of a lip, and parapets on Safe bridges. |

### Collapse compatibility

Rim links sit at outer-mid radius (about r 85–105) and spokes at r 32–48. A front moving inward removes **outer Shard edges, then rim links, then the Shards' inner parts, then spokes, then the Crown's terrace edge, then the dais.** A player on any Shard always has its own spoke until the Shard is gone, so nobody is stranded by a missing link. The graph is **radial in layers (rim links → spokes → Crown)**, which is the simple radial case.

### Recommendation: **Option B, Gated Pairs, typed**

- 6 spokes (3 Safe, 3 Risky), 3 Safe pair links, 1 Risky inter-pair link and 2 Dash-shortcut gaps: **10 bridges + 2 gaps**.
- Connector widths: **Safe 14–18 with parapets, Risky 8–10 with none, Dash gap 10–14 with 6-stud aprons.**
- Every Shard has its own Crown landing, none shared, and all six sit at least 40° from the Sentry lanes.
- Deliberate deviation: 3 of 6 neighbouring-region trips take about 12 s on foot (8 s with a Dash). Flagged above.

### **Settled (developer approved Option B, Gated Pairs, typed routes).**

- **All six Shards connect directly to the Crown**, one landing each at about **40°, 80°, 160°, 200°, 280° and 320°**. No two Shards share a landing. Each **Gate** has two arrivals around one rib: one **Safe** and one **Risky**.
- **Safe connector:** broad bridge with parapets, about **14–18** wide, viable during combat. Parapets should stop routine sideways Hook and Tidal deaths, and **the exact parapet height is validated in the greybox against the current Hook behaviour**.
- **Risky connector:** narrow exposed bridge, about **8–10** wide, no parapets, clearly readable as dangerous before committing. Tidal creates strong edge pressure but one tap is never an automatic kill. Sideways Hook kills are possible from exposed flanking positions.
- **Dash shortcut:** bridgeless gap, about **10–14** (provisional, pending movement validation), with clear takeoff and landing aprons, always with a non-Dash alternative. Spending Dash is the intended cost, and failing may mean a fall.
- **Jumpable gap:** not part of the required macro graph. It may exist later as local Shard geometry only after the jump distance is measured.
- **Outer ring:** the three pair links are **Safe** bridges. Of the three links between pairs, **one is a Risky bridge (210↔270) and two are Dash shortcuts.** No all-bridge outer ring, and no uniform ring.
- **Reasoning for the 12–13 s trip:** *The 8–9 second neighbour target is a general pacing target, not a requirement that every neighbouring Shard have identical foot travel. The two Dash-shortcut relationships are deliberately asymmetric: roughly 8 seconds if a player spends Dash, but roughly 12–13 seconds via the Crown if they preserve it. This makes Dash a meaningful rotation resource instead of merely faster walking.* The Dash gaps are **not** replaced with Risky bridges to preserve uniform travel time.
- **Principles:**
  - Every Shard always has at least one normal non-Dash route toward the rest of the playable Arena.
  - A player is never stranded solely because they conserved or spent Dash.
  - The Crown is never the only route between large portions of the Arena.
  - Risky routes create optional displacement combat, not mandatory death funnels.
  - Most fights happen on broad Shard and Crown terrain, not on bridges.
  - Safe and Risky connectors are visually readable before a player commits.
  - Route asymmetry is desirable as long as the map stays understandable.
- **Collapse compatibility:** outer and rim links can become unavailable first, each surviving Shard keeps its Crown spoke until that Shard itself becomes unsafe, and the Collapse never intentionally strands a player on a still-safe Shard.

---

## Point 5. Verticality

### Combat facts that decide the height language

| Fact | Consequence |
|---|---|
| **Basic Attack vertical allowance is ±5** studs, and **Arc Burst's is ±8** (configurable). Both use flat Sectors that ignore pitch. | A step or platform of **+4 or less** keeps melee and Arc Burst working across the edge. The **+5 dais is exactly at the melee limit**, so a terrace player may be unable to hit a dais-edge player in melee while Arc Burst still lands. That is probably fine as a small high-ground reward, but it has to be measured (it is not changed here). Anything above +8 turns Arc Burst off across levels entirely. |
| **Hook pulls flat in XZ, keeps the target's own height, and stops at static geometry.** | **A pull toward a higher caster is stopped by a step face** (the target is not lifted), so a Hook into a high-ground cliff does damage and no pull. **A pull toward a lower caster** carries the target out horizontally at its old height, and it drops to the lower ground. |
| **A pull across a gap toward a caster standing higher ends against the far cliff face, over the void.** | If the target is on a **lower** Shard and the caster on a **higher** one, the pull stops short at the face and the target falls. This kills without the caster being anywhere near a lip, which is a **cheap, near-unavoidable Hook kill** and the biggest reason not to use different Shard elevations. (It follows from the documented behaviour, and has to be confirmed in the greybox.) |
| **Tidal Bolt is a pitch-aware projectile** with a line-of-sight check, and Knockback is horizontal. | High ground helps sight and not much else. Knockback never lifts a target, so steps matter only as walls. |
| **Stone Guard is a Shield, not a directional guard.** | Verticality doesn't change it. A step face works as a retreat, not as a Stone Guard mechanic. |
| **Sentries have a clear-line-of-sight check, a locked aim and horizontal Knockback.** | Pedestals stay low (about 2–3). A Sentry above a lip pushes down and sideways, which is the same rule as Point 3. |
| **Monsters move directly toward targets where terrain allows, with pathfinding only when stuck** (`monsters.md`). | Any future Warden should live on flat ground or a ramped platform, and not in a terraced maze. |
| **Tidal Bolt's push distance conflicts between docs**: older notes in `studio-mcp-quirks.md` say 25 studs on a tap, and the later correction says about 5 (the rest was the target's own movement and falls). | I used about 5 in this design. **The greybox must measure it**, because the 5-vs-25 difference changes every lip margin on the map. |

### Consequences and failure modes

- **Whole Shards at different heights** make cross-gap Hook kills lopsided (above), turn "uphill" Shards into fortresses, and add a second variable to every spoke and bridge.
- **Stepped plates inside a Shard** work only if steps are 2 or less or are ramps. A 3-5 stud step is a wall for Hook pulls and a melee limit for Basic Attack, so it suppresses the exact play the map is meant to support, much as the old bays did.
- **High ground becomes dominant** when it has sight, reach and a safe retreat at once. A platform no taller than +4, with ramps on two or more sides, gives sight without a fortress.
- **Lower routes become traps** when they have a single exit, so there are no lower routes in v1. A depression is at most 2 deep and ramped on all sides.
- **Accidental falls from internal steps** are harmless as long as drops are small (a drop of 4 or less is, I'd expect, no damage, which the greybox confirms) and the **void plane is far below the lowest ground**. Falls into the void remain the only fall that kills. The kill-plane height itself is a configuration value, to be settled with the void boundaries.
- **Spectacle** is cheap to add later. Readable combat is hard to recover once a height language is built.

### Options

**A. Level Datum with Ruin Platforms (recommended).** Every Shard, every connector deck and the Crown terrace sit at **one datum, y = 0**. Terrain inside a Shard has gentle relief of ±2 at slopes under about 15°. Each Shard has **one or two ruin platforms of +3 to +4** with ramps on two or more sides. The Crown keeps only its locked +5 dais.
- Between Shards: **0** (edges that face each other are level to within 1).
- Crown height: 0, equal to the Shards, with the dais as its only rise.
- Connector slopes: **flat.** Bridges don't rise or fall, and arrivals are level.
- Basic Attack and Arc Burst: both work across every step. Hook: a pull is only blocked by a ruin face, never across a gap. Tidal Bolt and Arc Burst reads are clear.
- Fall readability: only lips matter. Warden: the dais and flat Shards suit it. Build cost: **lowest**, mostly flat plates plus a handful of ruin prefabs.
- Risk: may feel flat, so the ruins and the edge silhouettes carry the visual variety.

**B. Stepped Shards.** Each Shard has a high plate (+4–6) and a low plate (0) with ramps, and Shards sit at different datums within ±3.
- Between Shards: up to about 3, so cross-gap Hook kills become lopsided and bridges need slopes.
- Basic Attack: a +5 plate edge is at the limit, so melee on the low plate can't answer the high plate. Hook: ramps work and faces stop pulls.
- High ground dominates if the high plate has sight over the low plate and a ramp behind it.
- Cost: medium. Each Shard is two plates, ramps and per-connector level matching.

**C. Multi-level Shards (ziggurat).** Outer Shards lower, a rising terrace toward the Crown at about +12, Shards with ±8–12 of relief.
- Between levels: more than ±8, so Arc Burst can't hit across levels and Basic Attack barely can.
- Hook toward a higher caster never pulls, and toward a lower caster drops targets 8–12 studs. Cross-gap kills are lopsided by 8–12.
- Dramatic and readable from far away, but the high ground wins every fight. Lower routes are traps.
- Cost: highest, with sloped bridges, multiple datums and the Warden pathing problem.

### Where verticality comes from (the specific question)

**Structures and ruins on otherwise level Shards,** plus gentle terrain relief and the Crown's dais. Whole-Shard elevation differences are ruled out by the cross-gap Hook problem. Large internal terrain changes are limited to a depth of 2.

### Crown questions

- **Is the +5 dais enough?** Yes. With ribs at 8 high, it already gives the Crown three levels of cover (the terrace, the ribs and the dais). More height would only push melee and Arc Burst out of range.
- **Crown entrances:** **level.** The landing aprons, spokes and terrace are all at y = 0, so a Hook pull along a spoke is never stopped by a step.
- **Bridges crossing above or below another route:** **no.** Overpasses block projectile sight lines and make Hook, Tidal and readability worse. All connectors are coplanar.
- **Not in v1:** caves, ladders, multi-storey navigation, undercrofts and lower routes.

### Recommendation: **Option A, Level Datum with Ruin Platforms**

- **Datum y = 0** for all Shards, connector decks, aprons and the Crown terrace. Facing edges level within 1 stud.
- **Terrain relief ±2**, slopes under about 15°, no vertical step above 2 without a ramp.
- **Ruin platforms** of +3 to +4, one or two per Shard, about 14–20 studs across, with ramps on at least two sides and never abutting a lip within 10 studs.
- **Cover heights:** low cover about 3–4 and tall cover (the ribs) about 8. A ruin may block no more than about a third of its own Shard's lines.
- **Crown:** terrace at 0, dais +5 with ramps, ribs 8, Sentry pedestals 2–3.
- **No overpasses, caves or lower routes.**
- **Greybox measurements added to the list:** the smallest step that stops a Hook pull, the parapet height needed, the pull toward a lower caster, the Basic Attack and Arc Burst reach across a +4 and a +5 step, the tap Tidal Knockback distance and any fall damage.

### **Settled (developer approved Option A, Level Datum + Ruin Platforms).**

- **One movement datum, y = 0:** all Shards, connector decks, connector aprons and the Crown terrace. **No whole-Shard elevation differences**, and opposing connector edges stay on the same datum.
- **Terrain relief inside a Shard:** about ±2, gentle slopes generally under about 15°, and no abrupt internal step above about 2 without an intentional ramp.
- **Ruin platforms:** about **+3 to +4**, typically **1–2 per Shard**, about **14–20** across, at least **two accessible sides/ramps**, never within about **10 studs of a lethal lip**.
- **Crown:** dais about **+5**, ribs about **8** high as cover and line-of-sight blockers (not traversable platforms), spokes arrive level with the terrace.
- **Other heights:** low cover about **3–4**, Sentry/presentation pedestals about **2–3**.
- **Philosophy:** *Arena v1 gets vertical combat texture from small terrain variation and low ruin platforms, not from stacking entire combat regions at different elevations.*
- **Not in v1:** multi-level Shards, caves, ladders, undercrofts, a lower-route network, bridge overpasses, or one connector crossing above or below another.
- **Verticality must create sight-line and positioning choices without invalidating melee, Arc Burst or Rift Hook.**
- **Recorded Hook concern:** *Because Rift Hook preserves target height while pulling in XZ, meaningful elevation differences across gaps can create cheap asymmetric environmental kills or completely block pulls against cliff faces. Arena v1 avoids this by keeping opposing connector edges on the same datum.*
- **Greybox-validated, not assumed:** the smallest vertical obstruction that stops Rift Hook; the practical parapet height that blocks Hook/Tidal displacement; Hook behaviour toward a lower caster; melee reach across +4 and +5; Arc Burst reach and readability across +4 and +5; the actual Tidal Bolt tap knockback distance; whether standard fall damage exists or is disabled/irrelevant; and any interaction between the future void kill plane and vertical displacement. **If a measurement contradicts these values, adjust local geometry before redesigning the verticality model.**

---

## Point 6. Spawn layout and assignment

### What the code does today (read from `MatchRules`, `Match`)

- **`MatchRules.assignSpawns(order, pads, random)`** shuffles the pad list uniformly and gives the first N shuffled pads to the sorted player ids. It is pure, takes pads in whatever order `characters.padsIn("Map")` returns them, and wraps past the pad count.
- **Uniform random is the current policy, and it ignores geometry.** For 2 players on 6 ring pads: **40%** neighbouring Shards (about 8–12 s apart), **40%** two Shards apart, **20%** opposite (about 17 s).
- **Match start is one step.** `startMatch` builds the world, deals pads, spawns every player on a fresh character, then the phase is `Running` with the 480 s clock. **There is no spawn hold, no spawn protection and no GO.** The 15 s Countdown happens in the Hub, before anyone is on the map.
- The server-owned **Stunned / Frozen** gates already refuse casts, Dash and Pickups, but nothing sets them yet.

### Consequences and failure modes

- **Pad radius.** In Point 2 I put spawns at r ≈ 112, which leaves 18 studs to the outer lip. That breaks the "20 or more studs of margin" rule from Point 4. **Pads move to r ≈ 100**, which gives 30 studs of margin and keeps the walking targets (spawn → Crown about 4.3 s direct and about 5.5–6 s routed, neighbouring spawns 100 studs apart, opposite spawns 200 apart).
- **Lateral offsets are costly.** Moving a pad 20° toward either side of its Shard pulls one of its neighbours to about 54 studs: pair partners across a Safe bridge, or the Dash-shortcut neighbours across a 10–14 gap (a Dash would cross it in a couple of seconds). A pad exactly on a spoke line puts pair partners about 72 studs apart. Both defeat the Point 4 pacing.
- **The centre-azimuth pad is already offset from the spoke.** Shard centres are at 30°, 90°, … and landings at 40°, 80°, …, so every spoke starts about 10° (about 18 studs) off the pad's straight line. A sideways step is already built in.
- **Home loot and the spoke should not coincide.** If the first Caches sit on the line to the spoke, the "home route" and the "rush the Crown" route are the same walk. Point 7 puts the home Caches off that line.
- **Empty pads.** With 2–5 players, unused pads must not look like occupied ones or suggest a missing player.

### Pad placement options

**A. Outer-half, centre azimuth, facing inward (recommended).** Pad at the Shard's centre azimuth, **r ≈ 100** (about 30 studs from the outer lip), facing the Crown centre.
- First connector (the Safe pair bridge or the spoke): about **3.0–3.4 s**. Crown spoke start: about 3.3 s. Nearest lethal lip: **30 studs**.
- Neighbouring pad: 100 studs (about 6.3 s direct, about 8–8.5 s routed). Opposite pad: 200 studs (about 16–17 s routed).
- Sees: its own spoke and the Crown's silhouette. **No clear line of sight to any other pad** (a ruin or rise screens the rim side), so nobody sees a neighbour at spawn.
- No Sentry or Warden within 45 studs, since the Crown is 68 studs away. Environmental-kill risk near spawn: none by construction.

**B. Mid-Shard (r ≈ 89).**
- First connector about 2.4 s, Crown about 3.7 s direct (about 4.5 routed), neighbouring pads 89 apart (5.6 s), lip margin 41.
- Breaks the 6 s Crown target and the 8–9 s neighbour target, and it makes the Crown rush strong. **Reject.**

**C. Lateral offset (about ±20°, or landing-aligned).**
- A pad on the landing azimuth gives a straight spoke run and a Crown time of about 4.6 s. Pair partners end up 54–72 studs apart (3.4–4.5 s), and some pairs are within Hook, Dash and sight at the start.
- It also makes Safe-spoke Shards faster to the Crown than Risky-spoke ones, which makes the spawns unequal. **Reject.**

### Assignment policy options

**1. Keep uniform random (the current code).** No code change. Costs: 40% of 2-player Matches begin with the players one Shard apart, with empty Kits, about 8 s from contact. 3+ player Matches can cluster. Pacing and fairness depend on the draw.

**2. Spread table with a random rotation (recommended).** `assignSpawns` keeps its signature `(order, pads, random)`. Pads are listed **in ring order** (named `MapSpawn1`–`6` and returned sorted), and the function picks a **spread set** for N players, rotates it randomly, then deals the chosen pads to the players at random.

| Players | Chosen Shards | Meaning |
|---|---|---|
| 2 | one opposite pair (3 possible) | about 16–17 s apart, always |
| 3 | one alternating triple (2 possible) | no two adjacent, at least 173 studs apart |
| 4 | all but one opposite pair (3 possible) | two adjacent pairs, everyone else two apart |
| 5 | all but one random Shard | one empty Shard |
| 6 | all | every Shard |

- The rotation is random, so every Shard is used equally often and nobody has a fixed advantage.
- For 4 players, removing opposite Shards leaves two adjacent pairs. I'd leave the choice random, and not try to steer which kind of adjacency.
- **It's still pure, still a few lines, and the spec only grows.** The cost is losing "any pad could be anywhere", which matters little because players only learn the opponent's Shard.

**3. Hybrid: spread only for 2 and 3 players, uniform random for 4+.** Cheapest, but 4–5 player Matches then keep the worst-case clusters.

### Recommendation: **A (pad placement) with policy 2 (spread table)**

- Pad per Shard at the centre azimuth, **r ≈ 100**, facing the Crown, with a screen so pads don't see each other.
- Policy 2 for all counts. The 2-player opposite rule is the one that most improves the Match ("the Arena should work well at 2 players").
- **Preserving the existing code isn't a reason to prefer policy 1.** The pure function keeps its shape, and the real cost is one spec rewrite.

### Match-start fairness

- **Visibility of loot/PvE from spawn:** Caches are visible objects with unknown contents, which is already the rule. Sentries sit on the Crown, outside their 45-stud range from any pad. Pickups don't exist at the start. **No PvE with aggro or Home Territory within about 60 studs of any pad** (carried to Point 7).
- **Empty pads:** pad markers stay invisible setup data. A short ring effect marks only occupied pads, which fades at GO. Unused Shards show no pad.
- **Hold before GO (recommended): a 3 s Arrival hold.** Players spawn on their pads facing the Crown and see the map with a 3-2-1 countdown, but **can't move, cast or Dash until GO**. Nothing can reach anyone in 3 s from a spawn, so this isn't about damage protection. It is about **fairness across client load times, camera settling and reading the map**, which would otherwise cost a slow client its first seconds of a 480 s one-life Match.
  - **Constraints for implementation (the mechanism is the ticket's decision):** the 480 s clock and all pacing logs count from **GO**, so Ticket 20's numbers stay comparable. A leave or Eliminated during the hold behaves as in Running, and a hold may reuse an existing server-owned state (Stunned, Root) or a `canAct` gate. It changes nothing in the Hub.
  - **Risk:** if it needs a new phase it touches `MatchRules.step`, `checkEnd` and the gates, which Ticket 17 spent a lot of effort on. If so, the smaller form is a timestamp (`goAt`) inside Running that the gates read.
  - **Config:** `arrivalHoldSeconds`, with 0 meaning off, as a `Tuning.match` value.
- **No spawn protection** beyond that, and no invulnerability.

### Status of the two questions above: **superseded by Point 6A below, which must be settled first.**

---

## Point 6A. Designated spawns, free drop, or controlled Rift insertion

### What I checked

- **Match size is a real constraint.** `match-structure.md` uses a Match size of **8** as its batching example, and the design has six pads, one per Shard. `assignSpawns` wraps past the pad count today, which was explicitly "a limit of the current arena only". Anything above 6 players needs either more pads per Shard or a different insertion method. That is the honest scalability question, and it matters for A and C, less for B.
- **The Hub-to-Match flow is a single-server teleport-in-place today**, and the future outer loop is `TeleportService` into a fresh server. Clients will then load at different speeds, which is a second reason for an Arrival hold.
- **I couldn't read `Workspace.StreamingEnabled` through the MCP** (the property list came back empty). Whether the map streams in around a player is **unverified** and decides how safe any camera sweep or fall over distant geometry is. It has to be checked before presentation work.
- **Movement is client-controlled with server validation (ADR-0002).** Any airborne or gliding movement is a new category for Movement Validation, which currently explains movement through walking, Dash and Knockback.

### Option B, free drop: where it breaks (I tried to find the best case for it)

- **The Arena is too small for the drop to be a decision.** At a plausible transport speed of about 30 studs/s the whole Ø260 crossing is about 9 s. A usable jump window of 6–8 s then leaves seven landing zones. With 2 players the "strategy" is two people seeing each other jump, so it becomes a coin flip between "land near" and "land far". With 6 there is some spread, but still only seven zones.
- **There is an objectively best drop, and it's the Crown.** A central Cache gives two Abilities from one break (Point 3's rush problem). Walking, that's about 6 s away. A drop gets there in 2–3 s, so **B makes the centre-rush problem much worse**, not better, and Point 7 would have to solve it harder.
- **Everyone sees everyone.** With 2–6 players you can track every jumper on a 260-stud map, so the Shard-per-player opening disappears, and the likeliest outcome is a hot-drop on one Shard with an empty-Kit melee.
- **Late jumping.** Jumping late gives information (where everyone went) in exchange for time, which is a real advantage with so few players.
- **Cost and risk, all new:** a transport and its timeline, a glide or fall mode (custom movement, because Roblox has none built in), Movement Validation changes for airborne states, a server-authoritative landing, streaming of the landing zone, and a **void all around the Arena**, where a bad glide is an Elimination before the Match has begun. Each is a test matrix for a solo developer.
- **Where B does win:** it scales to larger Matches without adding pads, and it's the only option with real opening agency. Those are real advantages, but they only pay off at a population and map size this game doesn't have yet.

### Option A, designated spawns: what your list gets wrong, and right

- **Right:** fairness, controlled separation, a fair first Kit, loot-route balance, six territories, low cost, easy testing.
- **"Repeated Matches could develop predictable opening routes" is a smaller risk than it looks.** Route geometry repeats, but **what the home Caches hold doesn't**: the Ability bag is shuffled per Match, and the spread table rotates which Shards are used. The opening is "the same walk to different contents". Agency is the real loss, and I'd name it as the cost rather than minimising it.
- **A cheap middle option I'm not recommending:** let players pick among the Shards that remain during the Hub countdown (a draft). It gives agency, but it undoes the 2-player opposite rule, adds UI and a conflict-resolution path, and gains little at 2–6 players. Noted so it isn't forgotten.
- **Genuine weakness of A:** a hard ceiling of 6 players per Match. Going past it is a decision for when larger Matches exist, with likely answers being two pads per Shard (and a looser spread rule) or a larger Rift Crown.

### Option C, controlled Rift insertion: the one condition

C costs nothing mechanically **if it stays presentation.** The risk is that it grows into B by accident (a flight, a physical landing, a mid-air state). My condition:

- **The server places every character on its assigned pad first and holds it.** The "insertion" is then a client-side camera and VFX sequence over a character that is already in its final place, so there is no airborne state, no Movement Validation change, no landing authority and no void exposure.
- **It occupies the Arrival hold, it doesn't add to it.** If the presentation wants 4–5 s, `arrivalHoldSeconds` grows, and nothing else changes.
- **Its camera path stays inside geometry that is loaded for that client**, which depends on the streaming check above.

### Comparison

| | A. Designated | B. Free drop | C. Controlled insertion |
|---|---|---|---|
| Fun | Calm, consistent | Highest peak, but only with enough players | A's opening with a stronger first moment |
| Agency | None at the start | The only option with any | None, same as A |
| Fairness | High | Skill- and information-dependent | High |
| Replayability | Contents vary, geometry repeats | Highest | Same as A |
| Early pacing | A short hold, then loot | 6–8 s transport plus a landing | A's, with the hold stretched to about 4–5 s at most |
| Loot-route diversity | Six fixed, rotated | Free, and likely collapses to the Crown | Same as A |
| Crown-rush risk | Walking, about 6 s | **Much worse** (2–3 s) | Same as A |
| 2 players | Opposite, about 16–17 s apart | A coin flip over seven zones | Same as A |
| 6 players | One per Shard | Hot-drops likely | Same as A |
| Roblox complexity | Low | High (flight, glide, validation, streaming) | Low (client presentation) |
| Networking/physics risk | None new | New airborne authority and void exposure | None new |
| Solo-dev cost | Lowest | Highest | Low, and deferrable |
| Scaling beyond 6 players | Needs more pads | Scales naturally | As A, scatter-capable |

### Recommendation: **A mechanically, C as the later presentation, B not for v1**

I agree with your conclusion and for the reason you give, with two additions that I think matter more than the drop itself:

1. **B makes the centre-rush problem worse**, which is the strongest argument against it and wasn't in your list.
2. **The honest cost of A is the six-player ceiling**, not the loss of drop strategy. I'd record it as a known limit, so a Match size of 8 isn't assumed to work.

### Your eight items

| # | Item | My answer |
|---|---|---|
| 1 | **Pad placement A at r ≈ 100** | **Lock**, as a target. The real checks are in the greybox: at least 25–30 studs to any lip, no pad-to-pad line of sight, and every pad's first connector within 2–3.5 s. Irregular Shards will move individual pads by a few studs, and that's fine. |
| 2 | **Spread-table assignment with random rotation** | **Lock.** Add: the table covers 2–6 only, and above 6 the function refuses and logs, instead of silently wrapping (the old wrap becomes a bug at these sizes). |
| 3 | **3 s Arrival hold** | **Lock the value and the config** (`arrivalHoldSeconds`), and record that the later presentation may lengthen it to about 4–5 s. |
| 4 | **Clock begins at GO** | **Lock.** The 480 s clock and every pacing log count from GO. |
| 5 | **No movement, Basic Attack, Dash or casts before GO** | **Lock, and widen slightly:** also no Cache damage (it follows from no attacks), no Pickup collection or replacement, and no Sentry targeting before GO. **Allowed:** camera look, leaving to the Hub. |
| 6 | **Caches visible from spawn, contents unknown** | **Lock.** It's already the Ticket 18 rule. |
| 7 | **No aggro PvE/Home Territory within about 60 studs of an occupied spawn** | **Lock, but apply it to all six pads, occupied or not.** Occupancy changes with player count, the map doesn't, and a rule tied to occupancy would make 2-player Matches differ from 6-player ones. |
| 8 | **Occupied spawn presentation temporary, unused pads invisible** | **Lock, with the ring effect optional.** The pad markers are setup data. Any visible pad glyph is a presentation concern for the Match Intro ticket. |

### **Settled (developer approved A mechanically + C later as presentation, with the edits above). Points 6 and 6A are settled.**

**Spawn model**
- **Mechanics: designated Shard spawns.** A later **controlled Rift insertion** may visually deliver players to those positions, as **camera/VFX presentation only**.
- The server always places each character at its final spawn first and holds it until GO. No airborne gameplay state, no player-controlled landing, no glide movement category, no pre-match void exposure and no landing-authority problem.
- **Free-drop spawning is rejected for Arena v1.** Reasons: the Arena is too small for drop choice to create strategic depth; with only 2–6 players enemy drops are too easy to read; it makes the Crown-rush problem worse; it adds transport, glide, authority, streaming and void-failure complexity without enough gameplay return; and its advantages matter only with materially larger Arenas and populations.
- **Future hook:** *Free-drop insertion may be reconsidered if Riftfall later moves to significantly larger maps and match populations. It is not part of the assumptions for the first Rift Crown.*

**Pad placement:** six pads, one per Shard, centre azimuth, target **r ≈ 100**, facing the Crown. Greybox requirements: outer lip margin about **25–30+**, **no direct pad-to-pad sight line**, first connector reachable in about **2–3.5 s**. Local geometry may shift a pad if those conditions are better met.

**Spawn assignment:** the **spread table with random rotation**: 2 → opposite Shards, 3 → alternating, 4 → spread to avoid the worst clustering, 5 → all but one, 6 → all. The rotation is random, so no Shard is systematically preferred.
- **Capacity rule:** *The first Rift Crown supports at most six active Match players.* More than six at spawn assignment is **not wrapped onto occupied pads**. The assignment or Match start is refused, with clear diagnostic logging. Six is an **Arena-v1 limit**, not necessarily a permanent Riftfall limit.
- **No player-selected Shard drafting** in the Hub.
- *Implementation note (mine, not a design change):* refusing a start is a backstop. The roster should be capped at six when the Match is formed, with extra Hub players left for the next batch (consistent with the batching model in `match-structure.md`), or seven players in the Hub would mean no Match ever starts.

**Arrival hold:** `arrivalHoldSeconds`, initial **3**, **0 disables**. A future Match Intro may justify roughly 4–5, but presentation duration must **not** silently change gameplay timing without changing the configured value.
- **Disabled during the hold:** movement, Basic Attack, Dash, Ability casts, Cache damage, Pickup collection, Ability replacement, Sentry targeting, PvE aggro/combat interaction. **Allowed:** camera look and the normal leave/Return-to-Hub path.
- **At GO:** controls, combat and PvE become active and Match pacing begins.

**Match clock:** *The 480-second Match clock and all gameplay pacing telemetry begin at GO, not when characters are initially created or placed.*

**Spawn visibility:** nearby Caches may be visible from spawn with contents unknown. **No aggro-capable PvE/Home Territory within roughly 60 studs of any of the six pads**, occupied or not. (**Point 7 found a conflict with this rule, see below.**)

**Spawn visuals:** setup pads stay invisible during normal gameplay, unused pads never advertise empty spawn positions, any occupied-pad ring/glyph/VFX is optional, and polished visuals belong to the Match Intro ticket. *Spawn locations are gameplay data first. Their eventual visual presentation must not expose or constrain the underlying assignment system.*

---

## Point 7. Loot and PvE spatial slots

### What the code and docs fix (checked)

- **The bag is `Tuning.loot.pool` × equal copies.** Today 4 Abilities × 4 copies = **16 world drops**, all Rank 1, and equal counts are a strict invariant. The total must divide by the pool size. Early variety ("a pad's two nearest outer Caches differ") is **best effort** (`variety=partial` is allowed).
- **A Cache is 75 health outer (about 5 s of Basic Attack) and 120 central (about 8 s).** A central Cache drops two different Abilities about 10 studs apart. Contents are server-only until the break.
- **Rank 2 comes only from collecting a duplicate** of an owned Ability. There are two slots (Q, E), so the first two different Abilities fill the Kit.
- **Sentries:** range 45, a 0.5 s Telegraph, shots about 3 s apart, a locked aim, and a Tidal-Bolt hit with Knockback.
- **`match-structure.md` already describes the arc this point needs:** Gather (0–5 min, "getting everyone into a viable Kit"), then Clash (strong rewards pull players to contested areas). With one life and no respawns, the doc says a viable Kit has to come from "loot density and sensible spawn placement", with no catch-up mechanics.

### The supply problem, in numbers

The first thing the Shard layout exposes is that **the current 10 outer Caches don't fit six Shards**. 10 ÷ 6 is 1.67, so some Shards would hold two and some one, which is unfair at six players. And if the Crown is sealed for an opening period (below), then at **6 players the pre-Crown supply must be at least 12 drops** for everyone to reach a two-slot Kit without PvP. 10 isn't enough.

### Solving the deferred Crown-rush problem

| Solution | Verdict |
|---|---|
| **Timed seal on the Crown Caches** | **Recommended.** Removes the rush outright and is the smallest mechanism (one timestamp on a Cache record). It also turns the Crown's opening into a visible Gather → Clash event, which is what `match-structure.md` already describes. |
| **Leave unchanged** | Rejected. The rush yields a full Kit at about 14 s against about 25 s at home. |
| **Central Caches need a Sentry clear first** | Rejected. It makes PvE effectively mandatory, couples the rush to Sentry tuning, and a Kit-ready rusher clears it anyway. |
| **Central Caches hold upgrades only** | Rejected for now. It needs a new item type ("collectable only as an Upgrade") and Ticket 18 settled "no higher-Rank central loot". Once home supply guarantees a first Kit, the Crown's loot *de facto* becomes upgrade-and-replacement loot anyway. |
| **Fewer central drops (2 → 1)** | Rejected as the main fix. It lowers the prize but doesn't change the incentive to skip the home route, and it makes the Crown a worse objective. |
| **Move progression into Wardens** | Deferred. The dais is reserved, and Wardens come in their own ticket. |
| **Raise central health / Sentry rate** | Held as a tuning lever for evidence, not a design fix. |

**The seal:** central Caches can't be damaged until `crownOpenSeconds` of Match time (initial **60**, a band of about 45–90). The Sentries are **active from the start**, so rushing a sealed Crown costs health and gains nothing, and the sealed Caches look sealed from a distance. At the open, a visible and audible Rift event (a world effect, no new UI) announces it to everyone, which is also a deliberate noise cost. The time is a field on each Cache record, so a per-bay stagger is possible later without redesign.

### Slot types (the vocabulary)

| Slot | Meaning | Guarded |
|---|---|---|
| **Home Cache (H)** | Early, uncontested in practice, on the player's own Shard | No |
| **Forward Cache (F)** | On the Gate side of a Shard, toward the pair partner and the Crown | No |
| **Crown Cache (C)** | In a Crown bay, sealed until `crownOpenSeconds`, two drops | Sentry |
| **Warden slot (W)** | A Home Territory for a future normal Warden | n/a |
| **Elite slot (E)** | The dais, reserved for an elite Warden / high-value objective | n/a |

### Layout options

**L1. Ticket 18 redistributed (16 drops).** 6 home + 4 rim/gate Caches + 3 central × 2.
- Per Shard: 1 home, with 4 shared rim Caches. Not every Shard is equal.
- **Kit completion:** impossible without contesting. The pre-Crown supply is 10, so at **6 players two players have an empty slot until the Crown opens**. That violates the "viable Kit" aim and snowballs a one-life loss.
- Cost: lowest. Verdict: rejected.

**L2. Home Pair + Forward Cache (24 drops), recommended.** Per Shard **2 home + 1 forward = 18 outer Caches**, plus **3 central × 2 = 6**, a total of **24** (6 copies of each of 4 Abilities).
- **Every Shard is identical in count.** Pre-Crown supply is 18 (12 of them home), which covers 6 players' two-slot Kits with no PvP and leaves 6 forward Caches as the first contested layer.
- **24 divides by 4, 6, 8 and 12**, so a later pool of 6 or 8 Abilities needs no re-count (16 divides only by 4 and 8).
- **Deal rule change (needed):** each Shard's **home pair and its forward Cache hold three different Abilities** (a Shard misses one of the four). That is feasible with the equal bag (the central pairs take the remainder, e.g. counts 2, 2, 1, 1 form three distinct pairs) and makes "a full Kit without leaving your Shard" a guarantee, not a best-effort. **The forward Cache never duplicates the home pair, so Rank 2 always needs leaving the Shard.** That is how the Point 1 constraint (no self-contained safe island) is enforced in the loot.
- Cost: medium (more markers, a stricter deal, a bigger bag).

**L3. Home-lean, Crown-heavy (24 drops).** 12 home + 3 central × 4.
- Maximum convergence, but the Crown holds half the loot, which makes the rush incentive larger even sealed, and a 4-Pickup spread in one bay is cluttered. Verdict: rejected.

### Recommended layout (L2) in detail

| Slot | Count | Placement rule (coordinates fixed in the build pass, as Ticket 18 did) |
|---|---|---|
| **Home Cache** | 12 (2 per Shard) | 35–50 studs from its own pad (about 2.2–3.1 s), at least 15 studs closer to its own pad than to any other pad, at least 15 from any lip (10 margin + the ±5 drop spread), and not within 20° of the spoke line. The two home Caches are on different flanks of the pad. |
| **Forward Cache** | 6 (1 per Shard) | 60–75 studs from its own pad on the **Gate side** (toward the pair partner), so the two forward Caches of a pair flank the Safe bridge head. At least **50 studs from any Sentry**, so a Sentry never covers a Shard looter. About 4–4.7 s from the pad. |
| **Crown Cache** | 3 (in the bays at 0°, 120°, 240°) | As Point 3: r ≈ 19, Sentry r ≈ 27, ±5 spread, sealed until `crownOpenSeconds`. |
| **Sentry** | 3 (Crown only) | As Point 3. **No Shard Sentries in v1.** |
| **Normal Warden slot** | 3 | **One per Gate**, on the inner corner of the **Safe-spoke** Shard (30, 150, 270), about 55–65 from the Crown centre, next to the Safe spoke's start. At least 50 from any Sentry, on a ruin platform of about 14–20, with its Home Territory clear of connectors and lips. |
| **Elite slot** | 1 | The dais. (Sentry coverage of the dais is a deliberate overlap or a pedestal adjustment, to be decided in the Warden ticket.) |
| **Bridge objectives** | 0 | No Cache or PvE on a bridge or apron. Fights belong on broad ground. |

**Why Warden slots go on the Safe-spoke Shards:** the Safe spoke is parapeted and viable in combat, and the Risky spoke is exposed to the void. Putting PvE next to the Safe route and the void next to the Risky route gives each Gate one route that is dangerous because of PvE and one that is dangerous because of the edge. That is a hypothesis for the greybox, not a settled fact.

### Answers to the specific questions

- **Number of home slots, equal per Shard:** 2 home + 1 forward on **every** Shard.
- **Extra contested loot:** the forward Caches (6, at the Gates) and the Crown (6 drops).
- **Spawn to first progression:** about 2.2–3.1 s.
- **Full Q/E Kit without leaving the Shard:** **yes, guaranteed** (the home pair is two different Abilities), at about 25–35 s. The match docs want a viable Kit for everyone and no catch-up mechanic exists, so this is the right default.
- **Should that be allowed?** Yes, but **it can't stop there**: Rank 2 and a third Ability to choose from (the forward Cache) need the Gate, a neighbour, the Crown or a kill.
- **Rank 2 speed:** 2 players: about 60–80 s, by looting an empty neighbouring Shard. 6 players: mostly after the Crown opens (about 70–90 s) or from an Eliminated player's Kit drop. 4 players: in between.
- **PvE optional?** **Yes.** Nothing before the Crown requires killing anything. Sentries only guard the Crown, and Wardens are future and optional.
- **Noise/exposure per action:** a Cache break is a loud burst (audible nearby), a Sentry shot is a Telegraph plus Knockback, the Crown opening is a map-wide event, and a Warden fight is a visible Telegraph cloud. The loudest actions are the contested ones, which is what exposes a player to PvP.
- **Natural rotation:** at 2–5 players, empty Shards' loot, the forward Caches, the Crown opening and later the Wardens give reasons to move. At 6 players the Gate contest does it.
- **2 players:** opposite Shards about 17 s apart, Kits done by about 35 s, then they rotate to empty Shards, and the Crown opens at 60 s with both arriving with Kits. **6 players:** Kits done by about 35–45 s, partners meet at the Gate Caches, and six players converge on three bays at the open, which is the intended Clash (and the loudest moment of the Match).
- **Crown-rush incentive:** none before 60 s, because the Crown can't be looted and the Sentries are live. After 60 s the Crown is worth reaching with a Kit already, for Rank 2 and replacement choices.

### **Settled (developer approved L2, the 24-drop economy, the timed Crown seal and the amended PvE spawn-safety rule). Point 7 is settled.**

**Loot economy: 24 Ability drops** = **12 Home** (2 Home Caches per Shard, 1 Ability each) + **6 Forward** (1 per Shard, 1 Ability each) + **6 Crown** (3 Crown Caches, 2 Abilities each). This replaces the 16-drop economy.
- *Ticket 18's old Cache counts and Ticket 20's old-map pacing evidence are not expected to carry directly into the Rift Crown. The real Arena establishes a new progression baseline that must be measured independently.*
- The opening rule: *Every player can build a functional Q/E Kit on their starting Shard, but meaningful upgrades require interacting with the wider Arena.*

**Home progression:** 2 Home Caches per Shard at about **35–50 studs** from the pad (about 2.2–3.1 s), at least **15 studs from a lethal lip**, no Sentry guarding them, and no mandatory PvE for the first Kit. **The two Home Caches hold different Abilities, and the Forward Cache holds an Ability different from both.** The three local drops of a Shard form a **distinct Ability triple**.

**Full Kit guarantee:** a player may obtain a complete Q/E Kit without leaving their starting Shard. *A player cannot obtain Rank 2 purely from their own Shard*: the Forward Cache never duplicates a Home Ability. Rank progression needs another Shard, Crown loot, PvP Kit drops or later PvE. This preserves the rule that a Shard is never a self-contained safe island.

**Forward Caches:** 1 per Shard, about **60–75 studs** from the pad, toward the **Gate side**, so pair partners' Forward Caches pull players toward the shared Safe bridge region. **Never directly on a bridge, connector apron or lethal lip**, and about **50 studs or more from Crown Sentries** unless greybox testing shows a clearly safe alternative. Their purpose is to complete or broaden the opening Kit while moving the player toward contested space. They must not feel like a third Home Cache beside spawn.

**Crown seal:** `crownOpenSeconds`, initial **60**, expected tuning range **about 45–90**. Before it, the Crown Caches are **visible and their value readable, but cannot receive damage, open or drop loot**. At the open there is a clear world-visible and audible Rift event, so all surviving players understand the Crown objective is active without a hidden timer. **Each of the three Cache records owns its own activation timestamp/configuration**, so staggering is possible later. **No stagger in Arena v1** unless testing gives a reason.

**Crown-rush resolution:** *The Crown should be traversable before it becomes loot-active, but sprinting there immediately must not produce the fastest complete Kit.* The timed seal is the primary solution. Crown Sentries are active from GO, so an early rush gains positional information, rotation and possible PvP, but no immediate Cache reward, and it is exposed to Sentries. **Rejected or deferred:** stronger Cache health, mandatory Sentry clears, upgrade-only Crown items, reducing every Crown Cache to one Ability, mandatory Wardens.

**Crown Caches:** 3 bays, 1 Cache per bay, 2 Ability drops each. Their value makes the **Crown opening an important Clash event**. The Crown is not the required source of a player's first functional Kit.

**Sentries:** **3, Crown only**, one per bay, active from GO. **No Shard Sentries in Arena v1**, which keeps early progression readable and stops every starting territory from becoming a PvE encounter.

**PvE spawn-safety rule (replaces the fixed 60 studs):** *An aggro-capable PvE slot must be at least `aggroRange + spawnSafetyBuffer` from every spawn pad*, with `spawnSafetyBuffer ≈ 20` as the spatial planning value. The 25-stud aggro range originally used here as an example is **superseded**: [Ticket 23](milestone-2-tickets.md#23-rift-warden) ties `AggroRadius` to the Territory (`Territory + 4`, about 11 to 16), and at the +10° slot Home is only about 42 studs from its own pad, so aggro is capped near 22 by this rule. Also required: no direct spawn-line attack opportunity, and no Warden may aggro a player who has not intentionally advanced out of their immediate spawn/home area. If the Warden design needs a larger radius, **move the slot, never weaken spawn safety.**

**Future Warden slots (Reserved / hypothesis, not mechanically settled until the Warden ticket):** 3 normal Warden slots, one per Gate, hypothesised on the inner corner of the Safe-spoke Shards (about 30°, 150°, 270°), near the start of the Safe spoke and **not on the connector itself**. The experiment: *a Safe traversal route may carry PvE pressure, while a Risky route carries stronger void/displacement pressure.* If this makes Safe routes systematically worse than Risky routes, relocate the slots. **Settled by [Ticket 23](milestone-2-tickets.md#23-rift-warden) ([design](rift-warden-design.md)):** each slot carries a `GateId` and a `TerritoryRadius` and each pad a `GateId`. The preferred offset is +15 to +20° toward the Gate seam (about 48 and 66 studs to the own and partner pads), kept only if connector, apron, Territory, Forward Cache, lip, line-of-sight and ignoring-route checks pass, with the +10° slot as fallback. The Territory is a circle wholly on Layer 2 floor, at least about 15 studs from a lethal lip (the loot-safe margin), clear of collidable props, and **the Forward Cache must lie outside the Warden's automatic aggro**. If tuning aggro, placement, leash and reach cannot stop the Safe route being strictly worse, relocate the slot.

**Elite slot:** the Crown Dais, reserved for one future elite Warden or high-value objective. **No elite exists in Arena v1 merely because the slot does.**

**Bridge/objective rule:** *No Cache, Sentry, Warden or other required progression objective sits directly on a connector or connector apron.* Bridges are movement and combat decisions, not places to stand still and interact with loot.

**Opening flow (intended):** Spawn → first Home Cache → second Home Cache / choose route → Forward Cache → functional Q/E Kit → choose: rotate toward another Shard, hunt Rank progression, contest another player, investigate future Warden/PvE, move toward the Crown, or prepare for the Crown opening. **Target full-Kit time: about 25–35 s** (a playtest target, not a guarantee). At about 60 s the Match should move naturally from Gather toward Clash. At 6 players the open creates pressure around the three bays without requiring everyone to enter at once, and at 2 players it stays attractive without being the only rational next action.

**Rank 2 pacing is a hypothesis, not a rule:** about 60–80 s at 2 players and 70–90 s at 6. **Measure it.**

**Roster cap:** *Six-player capacity is enforced while forming the Match roster, not merely when assigning spawn pads.* The spawn refusal remains a backstop, and a seventh Hub player stays available for a later Match.

**Telemetry to add:** `CrownOpened match= seconds= alive= nearCrown=` (players within 70 studs of the centre at the open); `CacheBroken kind=Home|Forward|Crown` (extends `Outer|Central`); time to a full Kit and to Rank 2 per player; whether the first Crown break came from a player who arrived before or after the open.

---

## Point 8. Environmental kill geometry and the void boundary

### What exists today (checked in Studio and the code)

| Fact | Value / behaviour |
|---|---|
| `Workspace.Gravity` | 196.2 |
| `Workspace.FallenPartsDestroyHeight` | **-500** (the Roblox default), so an unhandled fall takes about 2.3 s and ends with the character removed |
| `Workspace.StreamingEnabled` | **false** (this settles the streaming question from Point 6A: the whole map is always loaded) |
| Jump | `CharacterJumpHeight` **7.2**, so a running jump at 16 studs/s travels about **8.7 studs** over an airtime of about 0.54 s on level ground (computed, to be verified). A 5-stud dais and a 4-stud ruin are both jumpable. |
| Walk speed | 16 |
| Fall damage | **None in the code** (only an animation reacts to free fall). Default Roblox characters take no fall damage, but this has to be confirmed in the greybox. |
| Death by a fall today | `Session.recordDeath` logs it as `cause=NotCombat`, `killerOf` returns nil, and `Match` already has `dropOrigin`, which falls back to `lastGrounded` for a player with no floor under them ("as after falling into the void"). The lifecycle for a void death is mostly there. |
| Where knockback is applied | `Combat` line 78 calls `movement.applyKnockback(hit.target, knockback, seconds)` with the hit and attacker in hand, which is the natural place to record "who displaced whom". |

**What doesn't exist:** an elimination cause for the void, any credit for a displacement kill, and a kill plane at a useful depth. At -500 a player would fall for 2.3 s before anything happens, which is too long to feel intentional.

**Two consequences of the new jump number for Point 2's bands:** a Dash of about 21 plus a jump of about 8.7 may carry a player over a **18–24 gap** if Dash works in the air, so the pair-link gaps (about 18–20) might be Dash-crossable and "bridge-required" would then be a soft label. That is harmless, but it blurs the Dash-shortcut gaps. If the greybox shows it, widen the pair-link gaps toward about 24 (still under the 30 limit) before changing anything else.

### Void/elimination models

**1. Immediate boundary elimination.** A hidden volume around and below the Arena kills on contact.
- Pros: instant and predictable, simple to implement.
- Cons: **the volume is a second description of the Arena's edge** that has to match irregular Shards and, later, every Collapse state. A player who Dashes or jumps across a gap would pass through it and die while physically safe. Nothing visibly happens at the moment of death, which is the opposite of "intentional and satisfying". Rejected.

**2. Physical fall + kill plane (recommended).** Players simply fall. When a living player's root passes `voidKillY`, the server eliminates them.
- The fall is real and visible, about 0.7 s from lip level to the plane (`voidKillY = -40`, about 43 studs below the root at lip height), long enough to see and for last-second actions, short enough not to drag.
- The Arena's edge is the **geometry itself**: no extra volumes, and it keeps working when the Collapse removes floor.
- Cost: one server check per Alive player on Heartbeat (at most 6), a new elimination cause, and the attribution rule below.
- The -500 setting stays as a safety net for anything that slips past.

**3. Fall state.** Leaving valid terrain starts a special state (locked controls, a countdown, or a recovery window) before elimination.
- Pros: dramatic, and allows rescue mechanics.
- Cons: a new state that has to interact with Charging, Dash, Knockback, Stun, Spectate and the Collapse. And **recovery is mostly fiction here**: the edges are sheer faces, so once the root is below lip level there is nothing to land on. The only real save is a Dash in the first fraction of a second, which the plain model already allows. Rejected.

### Recommendation: **Model 2, a physical fall + kill plane**

### The rules

**Where falling becomes death**
- **`voidKillY`: -40** (config), measured from the y = 0 datum. All Shards, decks and the Crown terrace sit at 0 (Point 5), so one plane serves the whole Arena. Ruin platforms and terrain never go below -2.
- **Instant on crossing the plane, with a visible fall before it.** The fall is the feedback: the player drops past the cliff face into the Rift for about 0.7 s, and a "swallowed by the Rift" effect plays where they pass the plane.
- **Applies only to Alive players of a Running Match on the map.** It doesn't apply in the Hub, the Reference arena (which has a deliberate `FloorLow` at -8), during the Arrival hold, or to Eliminated players. During Results, a falling survivor eliminates no one (the existing rule that a death in Results eliminates no one holds). To be confirmed in the specs.
- **A player may keep acting while falling.** There is no fall state, so a falling player can still aim and cast. A last-moment trade is allowed, and attacks in flight from an eliminated player already hit under Ticket 17's rules.

**Fall damage:** none expected. The greybox confirms it. If Roblox applies any, it is disabled, since a fall must never kill a player except through the kill plane.

**Recovery:** no special recovery. A Dash while the player is still above lip level can carry them back, and that is the only save. It is a skill window of a few tenths of a second, costs a Dash charge, and gets no coyote time or help. Whether Dash works in the air at all is measured, not assumed.

**Environmental kill attribution**
- **Credit goes to the last hostile displacement.** When `Combat` applies Knockback (Tidal Bolt, a Hook pull) to a player, it records `{ sourceId, time }` for the victim. If the victim is eliminated by the void, **the source gets the kill**, with Kill Recovery and the Match's kill count, in the same way as a direct hit.
- **The credit expires** when the victim has stood on solid ground for **1.0 s** (`voidCreditGroundedSeconds`), or after **6 s** (`voidCreditSeconds`), whichever comes first. This stops credit leaking onto a stroll to the edge long afterwards.
- **A walk or jump into the void unprovoked** is a **self-fall**: `Void` with no credit and no killer, still counted in the Match results as an elimination.
- **A Sentry's displacement** is logged against the Sentry and never credited to a player, as for Sentry hits today. (Point 3's Sentry-push rule means this should be rare.)
- **The Collapse later** can add a `source = Collapse` without changing any of this.
- **Cause:** a new elimination cause, `Void`, so results, logs and the Eliminated banner can say "fell into the Rift", with the credited player's name when there is one.

**Kit drops from a void death:** `dropOrigin` already falls back to the player's last grounded position. I'd add a minimum inward offset of **6 studs from any lip** so the loot never lands on the lip where collecting it is a gamble. This is my addition, and it can be dropped.

### Edge categories

| Category | Geometry | Where it belongs |
|---|---|---|
| **Lethal edge** | A **crisp, flat lip**, with no overhang, ledge grab or invisible wall, and a **sheer face** of 12 or more studs below it. The **last 6 studs of deck are the lip band**, in a distinct cracked Rift-stone trim that is readable in the first second. No parapet. | The outer perimeter of every Shard, the rim gaps between Shards, the Crown rim, the lips of Dash gaps and the sides of Risky bridges. This is the large majority of the Arena's edge. |
| **Safe edge** | Any change in height with solid floor within about 4 studs below it: terrain steps (≤2), ruin platform faces (+3 to +4), the dais face (+5) and ramps. Nothing here can eliminate a player. | Inside Shards and on the Crown (everything except the lethal rim). A player pulled or pushed over these lands on ground. |
| **Protected connector edge** | A **continuous solid parapet** along both sides of the deck, about **4 studs high and 1.5–2 thick** (provisional, to be validated), open at both ends (the aprons). It doubles as low cover. | **Safe connectors only** (three Safe spokes and the three Safe pair links). |

**Other geometry**
- **Risky connectors** have no parapet, so the deck is itself the lethal band. They read as dangerous through cracked, unrailed stone, and width stays at 8–10 (Point 4).
- **Dash-gap aprons** are 6 studs of flat deck before each lip, with the lip band marked, and nothing is placed on them.
- **The Crown rim and a Shard's outer lip use the same lethal-edge rule.** The Crown is simply where the density of fights near it is highest by design (Point 3), and the rest of its geometry keeps the centre safer than the rim.
- **No railings or walls anywhere else.** A continuous rail on the Shards' outer lip would remove the Arena's environmental-kill play, and broken ruin stubs near lips would only block Hook and Tidal for no gain.
- **The band width is a measured value, not a guess.** `lipBandStuds` starts at **6** and must be at least the strongest single-hit horizontal displacement among the Abilities (a Tidal tap is about 5, but the notes disagree about a Charged bolt, 8 or 40). The rule is: **a player standing outside the band cannot be killed by one hit, and one standing inside it can be killed by two.**

### Preventing cheap or accidental deaths

- **Margins already locked:** pads 25–30 or more from a lip, Home and Forward Caches 15 or more, ruin platforms 10 or more, Crown Caches and Sentry-covered standing spots 10 or more.
- **Hook:** a pull ends 4 studs from the caster, so a Hook kill needs a caster **at the lip with void between them and the target**. The caster stands in the lethal band too, so it is mutual risk. A Safe connector's parapet stops sideways pulls, and Risky connectors and Dash gaps are where Hook kills are meant to happen.
- **Tidal:** single-hit lethality is limited by the lip band rule above. Edge pressure comes from repeated hits and positioning, which is the intent.
- **Dash is the main accidental-fall vector** (about 21 studs in the player's chosen direction). I'm **not adding a Dash ledge guard in v1**, because Dash is a player-controlled movement and a guard changes combat. The mitigation is geometry (margins, the lip band, 6-stud aprons) plus telemetry. If `falls with a Dash in the last second` is high, a guard becomes a Ticket 21 follow-up decision, with evidence.
- **Walking off the edge is allowed.** There are no confirmation prompts and no invisible walls. Readability is the protection.
- **Spectator transition:** a void death takes the normal Eliminated path (the body, if any, is removed after `bodySeconds`, with the Kit dropped as in a normal elimination), and the spectate camera follows the existing rules. Any presentation of the fall belongs to the later Match Intro / presentation ticket.

### Rift Collapse compatibility

The model is **state-free geometry**: lethal means "no floor and below the plane", never a tagged volume. When the Collapse removes terrain, the new perimeter automatically becomes lethal edge, and the same plane eliminates anyone who falls. The Collapse only needs to restyle the new lip with the same trim and to add its own `source` for credit. No second void system is needed.

### Measurements (greybox, not assumed)

The tap and Charged Tidal Bolt knockback distances; the parapet height that stops Hook and Tidal pulls; Hook across gaps, with and without gravity during the pull; Dash length, and whether Dash works and keeps its distance in the air; running jump distance (computed about 8.7); fall damage (expected none); the real fall time to `voidKillY`; and the kill-plane check's latency.

### Telemetry

`Eliminated cause=Void credit=<id|none> via=Hook|Tidal|Sentry|Self creditSeconds= lastDashSeconds= zone=<nearest connector/edge marker>`; per Match `voidDeaths`, `creditedVoidDeaths`, `selfFalls` and `fallsAfterDash`.

### **Settled (developer approved Model 2, physical fall + kill plane). Point 8 is settled.**

**Void model:** *Physical fall first, server elimination at a configured kill plane.* Datum `y = 0`, **`voidKillY = -40` as the initial target, tunable after greybox testing**, `FallenPartsDestroyHeight = -500` stays only as a safety net. **No hidden edge-kill volumes and no dedicated Fall state in Arena v1.** The target feel is enough time to read that the player was knocked off, without a delayed or disconnected death.

**Scope:** applies only to **Alive** players in a **Running** Match on the Rift Crown map. It never triggers for Hub players, the Reference/Stage 3 environment, the Arrival hold, Results or non-Match characters.

**Falling behaviour:** no special movement state, automatic recovery, teleport-back, invisible safety wall, ledge grab, parachute or recovery prompt. A falling player may keep using whatever normal combat controls remain valid. *A very fast reaction Dash may save a player before they fall too far.* There is no special recovery Dash: it is the existing movement system interacting with the fall.

**Dash ledge guard:** **none in Arena v1.** Walking or Dashing off the map is allowed, with no automatic clamping. *Edge commitment is part of Riftfall combat. We should collect evidence about accidental Dash deaths before adding a system that changes movement behaviour.* Telemetry must be sufficient to measure accidental falls later, including an equivalent of `fallsAfterDash` (the exact representation is an implementation decision). No warning prompt, no invisible blocker and no special protection for accidental Dash use. If testing shows excessive self-falls, that is evidence for a follow-up, not a reason to add safety mechanics silently.

**Edge categories**
1. **Lethal edge:** open edge, no parapet, a crisp readable lip, a **sheer face of at least roughly 12 studs** before any geometry that could catch a player, and a visually distinct lip treatment. Locations: outer Shard perimeter, meaningful rim gaps, the Crown outer rim, Dash shortcut lips and Risky bridge sides. **The danger-band width is provisional, and 6 studs is not locked.** *The visual lethal-edge band should communicate the region where displacement becomes increasingly dangerous, and its final width should be tuned against the strongest intended single-hit displacement.* Measure the Tidal tap push, the charged Tidal push and any other displacement that can reach a lip. If charged Tidal is dramatically larger than assumed, **do not blindly widen every edge band**: re-grill whether the issue belongs to map geometry or combat tuning.
2. **Safe edge:** a vertical change that is not void danger (the dais face, ruin/platform drops, ramps, terrain steps, internal Shard transitions). Initial rule: solid floor within roughly **4 studs** below or adjacent (adjustable). It reads visually differently from a lethal lip.
3. **Protected connector edge:** **Safe connectors only.** A continuous parapet about **4 studs high** and **1.5–2 thick**, open at the aprons, doubling as low cover. Height and thickness stay provisional until Hook/Tidal testing. It prevents routine sideways environmental kills while still allowing normal combat.

**Railings:** none on ordinary lethal edges. No rails on Shards, the Crown, Risky bridges or Dash gaps.

**Risky bridges:** no parapet, an exposed deck, ordinary traversal remains possible, and displacement makes fighting there dangerous. They are not tuned so narrowly that one minor push is always lethal. Final width is subject to the displacement measurements.

**Environmental kill attribution:** a lightweight **displacement-credit record** is kept when a hostile player causes relevant displacement (Rift Hook, Knockback): attacker, timestamp and displacement source.
- **Expiry (starting values):** cleared after roughly **1 s of being grounded safely**, hard expiry after roughly **6 s** in total.
- A victim who falls because of recent valid hostile displacement: **the killer receives elimination credit, the death cause is `Void`**, and the source preserves enough to tell Hook from Knockback in telemetry.
- Walking, jumping or Dashing off with no hostile displacement: cause `Void`, **no killer credit**, treated as a self-fall.
- A Sentry push: cause `Void`, the source may record Sentry/environmental displacement, and **no player kill is awarded**. Future non-player PvE displacement follows the same principle unless a later ticket decides otherwise.
- **"Grounded" means genuinely returned to stable playable terrain**, not a one-frame contact with a cliff face, tiny geometry, a connector side or falling debris. A brief collision must not erase legitimate credit. The stable-ground test is an implementation decision.

**Kit drops after a void death:** 1) resolve the last valid grounded drop origin; 2) move it inward from the nearest lethal edge by about **6 studs** (an initial target, not a universal fixed vector); 3) validate it is on stable playable terrain; 4) if not, fall back to the nearest valid drop location, **never spawning loot into the void**. The goal: *environmental kills should reward the winner without making the dropped Kit unrecoverable or forcing another player onto the lethal lip to collect it.* The drop is not moved so far inward that it materially changes who controls the loot.

**Collapse separation:** `Void` means *leaving the physical Arena and falling below `voidKillY`*. The future Rift Collapse has its own damage/source semantics and may remove or consume terrain without any redesign of the base Void attribution/elimination.

**Greybox checks added by Point 8:** actual running jump distance; Dash distance on level ground; whether Dash chains with jumping or air movement; the practical maximum gap crossed by Dash + jump; actual Tidal tap and charged Tidal displacement; Hook across gaps; Hook against parapets; the minimum parapet height that reliably blocks displacement; fall duration from y = 0 to `voidKillY = -40`; whether any default fall damage occurs; whether a falling player can reasonably Dash back onto terrain; and whether connector widths produce the intended Safe/Risky distinction. **Revisit Point 2's connector gaps afterwards:** if Dash + jump reliably crosses the current 18–24 bridge-required gaps, **widen those specific gaps toward the upper end before redesigning the route graph.**

---

## Point 9. Rift Collapse compatibility of the Arena

*Scope: this point only makes sure Ticket 21's physical Arena supports a future Collapse cleanly. It does not decide damage, timings, speed, visuals, UI or overtime.*

### What the Arena already gives the Collapse

- **A radial structure by construction:** Shards at r ≈ 48–130, spokes at 32–48, the Crown inside r 32 with a dais at r < 14.
- **A void system that is geometry-derived** (Point 8), so any terrain the Collapse removes becomes lethal edge with no extra machinery.
- **Loot that already sits in layers:** pads and Home Caches in the outer Shard (Gather), Forward Caches, Wardens and pair links in the inner Shard, Crown Caches at r ≈ 19 (Clash). Collapsing outside-in therefore removes the Gather economy first and leaves the Clash prize for last, which gives **Gather → Clash → Collapse** with no extra design.
- **Locked principles to honour:** every Shard keeps its Crown spoke until that Shard becomes unsafe; the Collapse never strands a player on a still-safe Shard; the Crown is never the only route between large parts of the Arena; Risky routes are optional displacement combat, not death funnels.

### Failure modes the partition has to avoid

- **Stranding:** a section that disappears while its occupants have no route to a section that outlives it.
- **A permanent safe island:** one Shard that outlasts the others and lets a player hide (the Point 1 constraint, now applied in time).
- **Forced narrow crossings under pressure:** a stage that leaves a Risky connector as the only way in.
- **Reading difficulty:** with 6 players and a few seconds to understand, the order must be guessable from the map's shape.
- **Collapse removing the Arena's edge identity:** a removed section should leave a lethal edge with a readable face, not a thin slab.

### Partition models

**1. Simple radial layers.** Everything is partitioned by radius, in the user's order: rim links, then Shards, then spokes, then the Crown.
- Fair, because it is symmetric. Readable, because the Arena shrinks toward the middle. Anti-hide is strong (the outside always goes first). Replayability is lowest (identical every Match).
- Stranding: low, if the order is strictly inward. **Problem:** the Safe pair links sit between the layers, and removing all rim links before the Shards would leave Risky-spoke Shards with only their Risky spoke to retreat by.
- Complexity: lowest.

**2. Fragment-by-fragment.** Whole Shards become unsafe in a (random or fixed) sequence.
- **Unfair and hideable:** the Shard that goes last is a safe island for the whole middle of the Match, and a random order gives some spawns far longer than others. That breaks the Point 1 constraint in time.
- Stranding is high (a player on a falling Shard has a bridge or a spoke and nothing else), and a Shard's Gather and Clash loot disappears all at once.
- Readability is good (one Shard at a time), but anti-hide, fairness and the loot layering are all worse. Complexity is low to medium. **Rejected.**

**3. Hybrid.** Broad radial stages (as in model 1) with some randomness in which Shard goes first within a stage.
- It keeps model 1's fairness at the stage level and adds replayability. The risk is that the randomness could favour a spawn, and it must never reorder the inward chain.
- Complexity: low, since it is model 1 with per-section timing.

### Recommendation: **a radial layer partition built from per-Shard sections** (model 1's structure, with room for model 3's variation)

The map partition is **four layers, plus an optional fifth**, and every Shard is built as **two sections**, so ordering within a layer is a Collapse-ticket decision and not a map change. Ticket 21 builds the sections and the metadata. It does not decide any ordering variation.

| Layer | Contents | Approx. radius |
|---|---|---|
| **1. Outer** | The **outer section of each Shard** (pads, most Home Caches, the outer perimeter), the **Risky rim link (210↔270)** and the **Dash-gap pinches** | r ≈ 85–130 |
| **2. Middle** | The **inner section of each Shard** (Forward Caches, Warden platforms), and the **three Safe pair links** (they fall with the sections they connect) | r ≈ 48–85 |
| **3. Approach** | The **six spokes** and the **Crown's outer band** (landings, Sentry pedestals) | r ≈ 24–48 |
| **4. Inner Crown** | The **inner terrace**, ribs and Crown Caches | r ≈ 14–24 |
| **5. Dais (optional)** | The dais | r < 14 |

**Why the pair links go with the inner sections, not the outer layer:** it keeps the Safe route into the Crown alive for as long as the Shards that need it, and avoids the forced-Risky-spoke case above. The Risky rim link and the Dash shortcuts, which are the optional and exposed routes, go first.

### Answers to the specific questions

- **Whole or gradual?** **By section**, each unsafe as a unit after a telegraph that the Collapse ticket defines. A continuous radial shrink over irregular Shards would need runtime terrain cutting or fine tiling, and discrete states are easier to read. Any crumbling from the outside in during a section's telegraph is presentation.
- **Rim links vs Shards:** the Risky link and Dash gaps go with the outer sections (layer 1). The Safe pair links go with the inner sections (layer 2), after the outer sections.
- **Spokes:** layer 3, after the inner sections. **A spoke outlives its Shard's inner section**, so a player who is on it at that moment can finish the crossing.
- **No stranding, by construction:** every section has an **inward parent** that falls strictly later (outer → inner → spoke → Crown outer → inner terrace → dais). The routes between a section and its parent are solid ground inside one Shard, never a gap or a connector, with two rules: **section split lines don't run through ramps, ruins, Caches or Warden platforms**, and **a section's marker objects (pads, Caches, slots) all sit inside that section**.
- **Symmetry:** all six Shards go through the same layer together. **Variation in order within a layer is allowed in a later ticket** (per-section timing), as long as it never reorders the inward chain, and Ticket 21 builds nothing that forbids it. The default for Ticket 21 is simultaneous.
- **Forced narrow connectors:** a player who acts in time never has to cross a Risky connector. A Risky-spoke Shard has its Safe pair link until layer 2. A player who dithers past that point may be left with their Risky spoke, as a consequence of their own timing and not a surprise, and the crossing is only 16 studs.
- **Hidden region markers/volumes:** **none now.** Sections are real geometry models, so "unsafe" means the floor is gone, and the void system (Point 8) handles the rest. The Collapse ticket may derive warning zones from the sections' bounds.
- **A smaller final-fight area:** yes, **layer 4**, the inner terrace and dais of about **r ≤ 24 (about 1,800 studs²)**. That is about 900 per player in a duel and about 300 each for 6 survivors, so a 2-player duel, a 3-player fight and a pathological 5–6 survivors all fit.
- **The dais as the final location:** structurally **possible, not decided.** With the terrace gone, the dais is a Ø28 island (about 600 studs²) with lethal faces on every side, which is a pure displacement duel and cramped for more than 3 players. Whether it is ever the last safe ground is an overtime decision for the Collapse ticket. **Ticket 21 only has to make it possible**, which means the dais is built as a pillar (below).
- **Section thickness:** **every section is a solid slab at least about 12 studs thick, with a rock underside**, and the dais and ribs extend down into the terrace as a pillar. When a neighbour is removed, the face that remains is the lethal face from Point 8, not a thin sheet.

### Map metadata Ticket 21 should expose

- **Sections** (6 Shards × outer/inner, plus the Crown's outer band, inner terrace and dais) as separate models under one folder, each with a stable name, such as `Shard03.Outer`.
- **Attributes on each section:** `CollapseLayer` (1–5), `CollapseParent` (the section a player retreats to, nil for the last), and `ShardId` (or `Crown`).
- **Connectors** as their own models with `ConnectorType` (`SafeSpoke`, `RiskySpoke`, `SafePair`, `RiskyRim`) and `CollapseLayer`. **Dash gaps** as marker parts (no geometry) with the layer of the sections they sit on.
- **Slots:** every Cache, Sentry and Warden/Elite marker carries `SectionId`, so the Collapse can retire markers with their floor. What happens to Pickups lying on collapsing floor is the Collapse ticket's decision.
- **One `CollapseCenter` marker** at the Crown's centre for radial effects.
- **A check** (an execute_luau script or spec) that verifies the chain: every section's parent exists and has a strictly higher layer, and every marker's section exists. That is the acceptance check for this point.

### Requirements this puts on earlier points (to check, not to change)

- **Dash-gap pinches and the Risky rim link lie at r ≥ 85, and Safe pair links at r ≈ 55–85** (Forward Caches already sit there, so this matches Point 7's placement).
- **Pads at r ≈ 100 and most Home Caches fall in layer 1**, Forward Caches and Warden slots in layer 2, and Crown Caches in layer 4. A few Home Caches may fall near the split line, and the build pass should keep each Cache wholly on one side.
- **Crown Sentries (r ≈ 27) fall in layer 3**, so the final fight has no Sentries unless the Collapse ticket says otherwise. Flagged, not decided.

### **Settled (developer approved the radial layer partition and a structurally capable Dais). Point 9 is settled.**

**Direction:** *Radial authored layers, with optional future timing variation inside a layer.* **Fragment-by-fragment Collapse is rejected** as the Arena's base structural assumption (it favours spawn locations, creates temporary safe islands, undermines the radial progression and increases stranding risk).

**Five structural layers**
1. **Outer, r ≈ 85–130:** outer portions of all six Shards, spawn pads, most Home Cache positions, the outer perimeter, the Risky outer-ring bridge, the Dash-gap pinch regions.
2. **Middle, r ≈ 48–85:** inner portions of all six Shards, Forward Caches, reserved normal Warden platforms, the three Safe pair links. *The Safe pair links remain until Layer 2, because removing every rim connection before the Shards shrink would leave some Risky-spoke Shards with only their exposed Crown approach, turning the Collapse into a forced narrow-route problem.*
3. **Approach, r ≈ 24–48:** all six spokes, Crown landings, the Crown outer band, Sentry pedestals. The spokes outlive the Shard sections they serve.
4. **Inner Crown, r ≈ 14–24:** inner terrace, ribs, Crown Caches. The normal structural candidate for late-match combat. **About 1,800 studs² is accepted as the final ordinary combat region for Arena v1 planning.**
5. **Dais, r < 14:** the central Dais, marked **Structurally supported / optional for future Collapse use.** Ticket 21 does not decide that the Collapse will reduce the Arena to the Dais. A roughly Ø28 final island may suit a final duel, overtime or an anti-stall state, and is probably too small for an ordinary multi-player final stage.

**Inward-escape rule:** *A player's inward escape route must survive strictly longer than the section the player is being pushed out of.* A player who reacts to the telegraph always has solid inward ground.

**Behaviour the map supports:** *each physical section becomes unsafe as a unit.* The map is **not** built around a continuously shrinking mathematical circle cutting through arbitrary geometry. The Collapse may telegraph a section, make it unsafe and optionally crumble it. Crumbling visuals are presentation, not part of the playable boundary.

**Shard symmetry:** all six Shards use the same layer progression, Layer 1 before Layer 2 before Approach. No Shard skips ahead or stays an outer safe island. **Default: simultaneous within a layer**, with metadata allowing small timing offsets or per-section order jitter later, provided *variation may change timing within a layer but may never reorder the inward safety chain.*

**Parent rule:** *`CollapseParent` must exist and have a strictly later `CollapseLayer`.* The physical route from a section to its parent stays traversable while the section is active, never requires Dash, never requires jumping a lethal gap and never requires crossing a route that collapsed earlier. **Split boundaries avoid cutting through** ramps, ruin platforms, Caches, Warden slots, spawn pads and connector aprons.

**Narrow-route rule:** *The Collapse must not deliberately force an on-time player to cross a Risky connector.* A player who waits too long may be left with a dangerous route, which is acceptable because it is caused by delayed action.

**Geometry:** sections are **actual geometry**, not hidden region definitions: substantial slabs **about 12+ studs thick** with a readable rock/structure underside, so removing a neighbour naturally reveals a lethal face. The Dais and Ribs may extend down as structural pillars. No over-engineered destruction physics. Removing a neighbouring section must never expose paper-thin floating plates, visual holes that don't match collision, or non-lethal geometry below an intended void edge.

**Metadata**
- **Sections:** `SectionId`, `CollapseLayer`, `CollapseParent`, and `ShardId` where applicable.
- **Connectors:** `ConnectorType`, `CollapseLayer`, and the relevant section/Shard association.
- **Dash gaps** stay marker/data objects, not geometry pretending to be connectors.
- **Cache, Sentry and Warden markers** each reference a `SectionId`.
- **One authoritative `CollapseCenter` marker.**

**Structural validator (Ticket 21 acceptance check):** every `CollapseParent` exists and has a strictly later layer; every collapsible section has a unique valid `SectionId`; every Shard section has a valid `ShardId`; every Cache/Sentry/Warden marker references an existing section; every connector has valid Collapse metadata; and no required inward parent depends on a Dash-only route.

**Deliberately unresolved (left to the Collapse ticket)**
- **Crown Sentries** currently belong to Layer 3 and would disappear or go inactive before the Inner Crown fight. They are not moved to preserve them. Whether they disappear, deactivate, survive, transform or take part in a late Collapse is for the Collapse ticket.
- **Loose Pickups and Kit drops** resting on a collapsing section: whether they disappear, fall, relocate or stay collectible briefly is the Collapse ticket's decision. Ticket 21 only supplies the section ownership and position data to support it.

**Resulting flow:** Gather (outer Shards available) → Clash (outer terrain begins to disappear and players move through inner Shards and routes) → Collapse (approaches and the Crown become the remaining combat space), with no arbitrary late-game advantage for any spawn.

---

## Point 10. Visual and structural identity

*Scope: what this place physically is, so the greybox and the eventual build share an architectural language. No colours, textures, particles, sounds or uploaded assets.*

### Constraints this has to respect (from the settled points)

- The map is **authored geometry in `Workspace.MatchMap`**, because the environment queries (Hook sweep, projectile and Sentry line of sight, camera collision, floor raycasts) read **folder roots** through `Environment`. **Roblox Terrain is not in those roots**, so a terrain-based map would need engine-side changes to every query. Visible shape and collision must also match (Point 9).
- **Slabs 12+ studs thick** with a readable underside, **sheer lethal faces**, a **lethal-edge trim**, a **safe-edge look that differs from it**, **parapets only on Safe connectors**, **ruin platforms at +3 to +4**, **ribs 8 high**, **a +5 dais**, **sealed Crown Caches**, **Sentries on low pedestals**, **reserved Warden platforms**, and **sections that collapse as units**.
- **Ability colours already exist and must not be confused with the world:** Tidal blue, Arc Burst lightning, Rift Hook amber-orange, Stone Guard stone grey with amber. The Rift seam and the world's trim must read as different from all four.
- No asset is uploaded without asking, so the first pass uses Roblox's built-in parts and materials.

### Options

**A. Shattered ancient arena.** A purpose-built combat or ritual arena that the Rift broke apart.
- **Why six Shards and a Crown:** six stations around a central ring, such as seating wards and a floor.
- **Bridges, ribs, dais:** causeways between stations, the remains of a dome's arches, a champion's platform.
- **Edges:** broken floor, the open sky as the "missing" arena. Safe/Risky: wide processional ways against collapsed walkways.
- **Caches, Sentries, Wardens:** prize coffers, arena guardians, arena keepers. **The seal:** the arena's gate.
- **Collapse:** the foundations fail.
- **Readability:** a ring of stations is legible from spawn.
- **Cost:** medium to high, because seating, rings and gates are bespoke shapes. **Identity:** weak. A Roblox "gladiator arena" is a very common idea, and it explains the Match more than it explains Riftfall.

**B. Ruined Rift citadel (the shattered crown).** An older monumental structure, a crown-shaped citadel, that was never an arena and has been fractured by the Rift into its present form.
- **Why six Shards and a Crown:** the name does the work. **The six Shards are the broken points of a crown, and the Crown is its circlet,** with the Dais as the empty setting where something once sat. The structure explains the topology and the title with one image.
- **Bridges, ribs, dais:** the **ribs are the surviving arches** of the crown's spires, the **Dais is the jewel seat** (an empty throne or socket, which is why it can later hold an elite), and the **bridges are the causeways and chains that once joined the points to the circlet.**
- **Open void edges look intentional:** a structure floating in the Rift has no "outside wall". The edge is where the masonry ends, and the exposed underside is the structure's own rock. Lethal edges carry a glowing **Rift seam**, because the Rift is what cut it.
- **Safe vs Risky connectors:** **Safe** = a broad, worn causeway with heavy masonry parapets and iron bands. **Risky** = a narrow, cracked span with missing chunks, no sides and loose chains hanging beneath. **Dash gaps** = a **bridge that broke in the middle**, two stubs facing each other with chain ends and paved aprons.
- **Ruin platforms (+3/+4):** collapsed halls and foundations, masonry mounds with ramps.
- **Caches:** crystal-veined stone reliquaries set into ruins (the Cache look already uses cracks that glow).
- **Sentries:** carved **guardian constructs** on low pedestals. **Wardens:** the "Stone/Rift" Monster Family in `monsters.md`, and the **name Rift Warden** already means a keeper of something.
- **The Crown seal:** the Crown Caches are **bound by chains and a glyph ring**. At the open the chains snap and the glyph shatters, with the horn the open event already needs.
- **Collapse:** the Rift consumes the monument from the outside in, along **pre-fractured section seams** that are visible in the floor, so the layers are learnable before they matter.
- **Silhouette from spawn:** every pad faces inward at **eight-stud arches and a raised dais** that read as the points of a crown.
- **Distinguishing Shards without six biomes:** **one signature landmark per Shard** (a broken obelisk, an arch, a stair, a brazier ring, a statue plinth, a toppled column), built from the same kit and materials.
- **Cost:** low to medium, because it is a **small modular vocabulary** reused everywhere. **Identity:** strongest. It is specific to this game's title and combat.

**C. Natural floating Rift landscape.** Mostly rock and crystal terrain with ruins as a secondary layer.
- **Why six and a Crown:** a shattered landmass. **Edges:** natural rock.
- **Risks:** organic, irregular edges **blur the lethal lip and the safe edge**, exactly where Point 8 needs crisp readability. Organic rock is not easy to hold to the 12-stud-slab and section rules, and as noted, Terrain would not be seen by the environment queries.
- **Bridges, ribs and dais** have no reason to exist, so they need added ruins, which makes the ruins the real identity anyway.
- **Cost:** the lowest per piece but the least modular. **Identity:** generic floating island.

### Comparison

| | A. Shattered arena | **B. Ruined Rift citadel** | C. Natural landscape |
|---|---|---|---|
| Explains six Shards + Crown | A ring of stations | **A broken crown: the title itself** | A broken landmass |
| Explains bridges, ribs, dais | Partly | **Yes, each one** | No, needs ruins |
| Edge readability | Good | **Good** (seam trim, sheer face) | Weak |
| Safe vs Risky readability | Good | **Good** (parapet, iron bands vs cracked span, no sides) | Weak |
| Sentries, Wardens, seal | Fit | **Fit** (constructs, keepers, chains and glyph) | Awkward |
| Collapse reads naturally | Partly | **Yes** (pre-fractured seams) | Hard (organic) |
| Fits the Environment queries | Yes | **Yes** | Needs engine-side changes if Terrain |
| Modular for one developer | Medium | **High** | Low |
| Identity | Generic | **Strongest** | Generic |

### Recommendation: **B, the ruined Rift citadel (the shattered crown)**

It is the only option in which every settled structural element has a reason to exist, and it costs the least to build because it is a small kit.

### Structural kit (a vocabulary, not assets)

1. **Slab:** thick dark stone decks (12+ studs) with a **chamfered lip** and a **faceted rock underside**. Every Shard section, spoke, bridge and Crown piece is a slab.
2. **Rift seam:** a glowing crack that runs along **lethal edges only**. It is the single visual signal for "this edge kills". It is also used for the Crown seal glyph, and for the Cache cracks that already exist. **Its hue must not be Tidal blue, Arc lightning, Hook amber-orange or Stone Guard grey/amber.**
3. **Masonry:** broken walls, steps and ramps. Used for ruin platforms (+3 to +4), low cover (3–4) and tall cover (8).
4. **Column stumps and shafts:** broken columns as cover and as the ruin vocabulary.
5. **Arch/rib segment:** the signature shape. Three at the Crown (the ribs), and **one per Shard as its landmark**.
6. **Iron bands and chains:** dark metal straps and hanging chains. They mean "held together": heavy bands on Safe connectors, **loose hanging chains on Risky ones**, snapped chain ends on Dash-gap stubs, and binding chains on sealed Caches. **Not amber**, which belongs to Rift Hook.
7. **Parapet:** the solid masonry wall (about 4 high) used **only** on Safe connectors.
8. **Plinth:** a low carved base. Used for Sentry pedestals (2–3 high) and the Dais.
9. **Paving:** flat, worn stone for aprons, so the places where nothing may be placed read as approaches.
10. **Crystal clusters:** a few small, mostly non-colliding, accents in the seam hue.

**Rules that make the vocabulary readable**
- **Lethal edge = seam trim.** **Safe edge = plain chamfer or steps, with no seam.** **Protected edge = masonry parapet with iron bands.** A player should be able to tell all three apart at a glance.
- **Risky decks carry no geometry at their sides** (no stumps, rails or posts), because a Risky deck is the lethal band and anything at its edge would block Hook and Tidal pulls. They read as risky through cracks, missing chunks and hanging chains.
- **Collision matches the visible shape** everywhere, and nothing decorative that a pull or projectile can hit sits at a lip.
- **Section seams are visible** in the floor (the pre-fractured lines), at the section boundaries from Point 9.
- **One landmark per Shard**, from the same kit and materials. No separate biomes.

### What Ticket 21's greybox must already carry

The **slab thickness and sheer face, the lip trim as a strip on lethal edges, the parapet on Safe connectors, the Risky deck with no side geometry, the aprons, the ribs, the dais as a stepped plinth, the Sentry plinths, a seal object on each Crown Cache, a landmark placeholder per Shard, and the section seams.** Everything else (exact colours, textures, particles, sounds, meshes and uploads) belongs to a later art pass, and any upload is asked about first.

### **Settled (developer approved identity B, the ruined Rift citadel / the Shattered Crown). Point 10 is settled.**

- **Identity:** a ruined citadel shaped like a shattered crown, suspended over the Rift. Six Shards are the broken points, the Crown is the surviving circlet, the Dais is the empty jewel setting or throne. Generic gladiator arena and mostly-natural floating landscape are rejected for v1.
- **Ten-piece kit locked:** slab (12+ thick, chamfered playable lip, faceted underside); Rift seam (lethal edges and Collapse-readable fractures only, never generic decoration); masonry; column stumps; arch/rib segment (signature silhouette, reused for Crown ribs and Shard landmarks); iron bands/chains (visually distinct from Hook amber-orange); parapet (Safe connectors only); plinth (Sentry pedestals, Dais, future Warden/objective bases); paving/apron (connector arrivals, Gate transitions); crystal clusters (sparingly).
- **Edge readability locked:** Lethal edge = open lip, visible Rift seam, no parapet, readable sheer drop, no side geometry that could block Hook/Tidal. Safe edge = plain chamfer, masonry step, ramp or internal transition, never seam treatment. Protected connector edge = masonry parapet with iron reinforcement, continuous. Risky connector = fully exposed sides, cracked open deck, no side geometry, and nothing hanging beneath with collision (chains below are non-blocking presentation).
- **Crown seal:** each Crown Cache visibly bound (chains + glyph/seam). The map supplies the objects and anchors only. Final VFX/audio is not in Ticket 21.
- **Sentries and Wardens:** Sentries read as built/carved guardian constructs of the citadel. Wardens belong to the same Stone/Rift family without being confusable with Sentries. No monster appearance is finalised here. PvE should look native to the Shattered Crown.
- **Shard distinction:** one signature landmark per Shard from the same kit. No separate biomes. Differences come from arch arrangement, column clusters, masonry silhouettes, platform shapes, broken crown-point geometry and local ruin composition.
- **Ability-colour separation:** environment does not lean on Tidal blue, Arc lightning, Hook amber-orange or Stone Guard grey/amber, and never resembles active Hook VFX, Ability pickups or Reaction indicators. Exact palette deferred to the art/UI pass.
- **Authored geometry rule:** Arena v1 is authored Part/Model geometry under `Workspace.MatchMap`. No Roblox Terrain.
- **Collision/readability rule:** visible geometry and collision agree. No invisible blockers at lethal edges, no decorative walls without collision, no oversized collision proxies, no chains/crystals that catch Hook pulls, no visual openings that are blocked. Gameplay readability beats visual detail.
- **Section seams:** visible authored seams, subtle enough not to look like grid lines, not hidden. The later Collapse should read as the world following existing fracture lines.
- **Greybox must carry:** slab thickness, lethal lip trim, Safe parapets, Risky exposed decks, connector aprons, Crown ribs, Dais plinth, Sentry plinths, one seal object/anchor per Crown Cache, one landmark placeholder per Shard, visible section seams, and underside geometry sufficient to make removed sections believable. No final colours, textures, meshes, particles, sounds or uploads (no upload without explicit approval).

---

## Point 11. Arena v1 active environmental hazards

*Scope: whether the first real Rift Crown has any active map hazard beyond void, lethal edges, Sentries, future Wardens and the future Collapse. Principle: **Arena hazards should create decisions for players, not random deaths.***

### What the Arena already has

Void and lethal edges (Point 8), Safe/Risky/Dash connectors (Point 4), three Crown Sentries active from GO (Point 7), the timed Crown seal (Point 7), reserved Warden slots, and the Collapse (Point 9, designed later). That is already four pressure sources, and two of them (Sentry pushes, Collapse) do displacement or terrain change.

### Why this question matters

Every active hazard has to answer three things the combat does not already answer: who gets blamed for a death (kill credit), whether it competes with Hook/Tidal for the same environmental kill, and whether a new player can read it in the first match. The displacement-credit rule from Point 8 covers *player-caused* pushes. A hazard that pushes or kills on its own adds a third cause (the map) with no player to credit.

### Options

**A. No additional active hazard.** Void, lethal edges, Sentries, future Wardens and the Collapse only. All other geometry is static.

**B. One light authored hazard family**, for example a telegraphed Rift vent that fires a knockback pulse, a crystal eruption, or a temporary local hazard zone.

**C. Dynamic traversal hazard**: collapsing bridge, disappearing floor, moving platform or cover.

| | A. None | B. One light hazard | C. Dynamic traversal |
|---|---|---|---|
| Improves PvP | Keeps the fight about Hook, Tidal and positioning | Adds a position to contest, if telegraphed | Adds timing mind-games, but of the map, not of players |
| Fair environmental kills | Yes, every void death has a player cause or a clear own mistake | A vent push is a third displacement source. Credit is unclear unless it is "no player kill" | Worst: deaths with no actor |
| Competes with Hook/Tidal | No | **Yes.** A shove that beats a Tidal tap muddies who killed whom | Yes, and it changes which lines exist |
| New-player readability | Highest | Medium, needs a learned telegraph | Low, the map changes under their feet |
| One-life interaction | Clean | A random-feeling loss costs the whole Match and drops the Kit | Same, worse |
| Future Collapse | Collapse is the one terrain change, so it stays legible | Two systems telegraph things on the floor | Collapse and moving floor fight each other, and the structural validator (Point 9) assumes static sections |
| Sentry/Warden overlap | None | Sentry push + vent push stack. Warden fights near a vent get unreadable | Warden slots on moving ground |
| Scripting | None | A timer, a zone, a telegraph, a server push, credit rules | Server-moved geometry, replication, sync with Hook sweeps and floor raycasts |
| Testing burden | None | Per-vent timing, credit, spawn-safety specs | Large: physics, desync, stuck players |
| Failure modes | The map feels static mid-Match (a pacing risk, not a bug) | Vent kills on a Risky route become the dominant route-avoidance reason | Players stuck inside moving geometry, false Hook hits |
| Solo-dev cost | Zero | Medium | High |
| Enough complexity already? | **Yes** | Marginal | No |

### Recommendation: A, no additional active hazard in Arena v1

**Reasons**
1. The Arena's selling point is that **players** cause environmental kills with Hook and Tidal. A hazard that also kills or pushes dilutes that and creates deaths with no player behind them.
2. Pressure is already layered: Sentries (early, Crown), Wardens (future), the Collapse (late). Adding a fourth source before we have measured the first three is guessing.
3. The Collapse is the designed late-game terrain change. A second one makes both harder to read and breaks the static-section assumption the structural validator relies on.
4. The map is too new to know what is missing. If the greybox play-test shows a dead mid-game, we add the hazard with evidence, not in advance.
5. Solo-dev cost and testing burden are zero.

**The pacing risk, stated honestly.** Without the Collapse (not in Ticket 21) and without a hazard, a Match can stall in a stand-off. A hazard is the wrong cure: the cure is the Collapse and Warden pressure. Ticket 21 records this as a play-test observation (idle-time and Match-length telemetry), not a reason to add a hazard.

### What stays reserved (not in v1, not rejected forever)

- **Rift vent / knockback pulse:** reserved for a later Arena variant or a post-Collapse tuning ticket. If ever added: telegraph of at least 1 s with a ground marker, a damage-free displacement only, never on a Safe or Risky connector, never within 8 studs of a lethal lip, never within spawn-safety range of a pad or a Sentry plinth, a push that follows Point 8's "no player kill credit" rule unless a player caused it.
- **Crystal eruption / temporary hazard zone:** same constraints, plus no direct damage that can eliminate.
- **Collapsing bridge, disappearing floor, moving platform or cover:** reserved for the Collapse ticket or a dedicated Arena variant. Not in Arena v1 because the Collapse already owns terrain change.
- **Hazard anchor hook:** the greybox does not carry hazard anchors. Adding a hazard later means adding its own marker folder and server module, without changing the Shard/section structure.

### Direct-elimination and credit rules for v1

- **Environment can directly eliminate a player only through the void** (Point 8). No hazard deals damage.
- **Kill credit** stays exactly as Point 8 set it: displacement credit from Hook/Tidal within the window, none from Sentry pushes, none from the Collapse.

### **Settled (developer approved Option A, no additional active hazard in Arena v1). Point 11 is settled.**

- **Arena v1 environmental pressure is only:** void and lethal edges, Crown Sentries, future Rift Wardens, and the future Rift Collapse. No vents, knockback pulses, crystal eruptions, temporary damage zones, collapsing bridges, disappearing floors, moving platforms or moving cover.
- **Principle:** environmental kills in Arena v1 come primarily from player positioning and player-caused displacement, not from the map killing players independently. The Collapse is the intended dynamic terrain-pressure system, and Arena v1 introduces no second terrain-changing system before it is tested.
- **No map hazard deals damage.** The environment eliminates a player only through the existing Void system. Sentries and PvE are combat actors, not map hazards.
- **Reserved (future candidates, not rejected):** Rift vent / knockback pulse (telegraph of roughly 1 s or more, visible ground marker, displacement only, never on Safe or Risky connectors, within roughly 8 studs of a lethal lip, within spawn-safety range or beside a Sentry plinth; kill credit only if a later design can establish a player caused it, and no attribution system is invented now). Crystal eruption / temporary zone (strongly telegraphed, away from connectors, spawns and stacked lethal edges, must not deal elimination-level damage in one-life combat, needs its own ticket).
- **Moved out of Ticket 21:** dynamic traversal hazards belong to the Collapse design, a later Arena variant or a dedicated hazard ticket. No placeholder scripts or hazard anchors.
- **No generic hazard-marker framework.** A later hazard brings its own marker/data model and references `ShardId`, `SectionId` and the Collapse structure.
- **Pacing risk recorded:** before the Collapse exists the Arena may allow stand-offs or inactivity. Not solved with hazards. Ticket 21 captures evidence (see its telemetry section).
- **Final principle:** Arena v1 first proves topology, loot routes, PvE pressure, displacement combat and the Collapse are enough. Additional hazards earn their complexity through play-test evidence.

---

All eleven points are settled. The Ticket 21 entry in `milestone-2-tickets.md` is assembled from this document.

---

## Review amendments (2026-11-09, after the adversarial review of the ticket)

**Status: all amendments confirmed by the developer on 2026-11-09.** Items 1 and 2 are now locked rules (the Forward Cache structural rule with 50–58 as a tunable target only, and the Sentry buttress with its requirements and sweep check). The build/swap strategy is locked as written in the ticket.

Changes made to the Ticket 21 entry that go beyond copying settled decisions. Flagged here so none is mistaken for something you already approved.

1. **Forward Cache distance.** The settled 60–75 studs from the pad does not fit an about 80 × 82 Shard with a 15-stud lip margin (the farthest valid point toward the Gate is about 50–58). The ticket keeps the settled intent as the rule ("clearly farther than Home, toward the Gate") and 50–58 as a tunable start. **Confirmed.**
2. **Sentry buttress.** A Sentry at r 27 is 5 studs from the lip, so a player behind it would be pushed over. The ticket adds a masonry buttress filling the rim band behind and beside each Sentry, with a sweep check. **Confirmed.**
3. **Ability deal.** Settled as invariants, with a verified reference algorithm (each Shard omits one Ability, omissions balanced {2,2,1,1} and randomised per Match over Abilities and Shards, the Crown gets the omitted copies, no Crown Cache holds a duplicate).
4. **Roster cap versus backstop**, **build and swap strategy**, **seal and hold ownership**, **void attribution failure table**, **simplified Kit-drop relocation** (inward direction from the exit point, with spread points validated), the **measurements table** (expected value, threshold and decision separated), and the **part-count baseline** were added or tightened.
5. **Clarified:** the Inner Crown's about 1,800 studs² is the disc r ≤ 24 including the Dais top. The terrace ring alone is about 1,190.
6. **Glossary:** tunable numbers removed from the Dais, Safe connector, Risky connector and Dash shortcut entries, and Crown Cache added.
