# Match Milestones: settled design

The reasoning behind [Ticket 25](milestone-2-tickets.md#25-match-milestones) and [ADR-0009](adr/0009-match-progression-through-one-time-milestones.md). The ticket is the build contract and holds the rules. This file holds why. Every point was grilled with the developer on 2026-11-10, one at a time.

**Terminology.** A **Milestone** is a one-time reward earned by a qualifying feat. A **Lane** is one of three kinds of feat: **Explore**, **Contest** and **Endure**. A Milestone is not a Rank, a level or XP. The rewards are **Reserve**, **Riposte** and **Catch Breath**.

**The status.** Specified, not built. The design is deliberately **conditional on gameplay evidence**: it is built in stages, and each stage waits for the evidence recorded in the ticket.

## Point 1. Purpose and structure

**Settled: a small tactical layer on top of the Kit, not a power curve.** Ability Ranks are the only Ability upgrade system, and the old-map floor pacing (about 7.8 s to a first Ability, about 17 s to a full Kit, Ticket 20) shows a Kit stops changing early. That evidence is from the old map, so the Rift Crown must be measured again. Milestones give a player something to pursue and decide about afterwards.

- **Considered:** no progression (telemetry only), XP with levels, and bounded Match progression. XP was rejected for farming and snowball risk, and for a second power system next to Rank. Telemetry-only stays the fallback.
- **Structure:** one-time Milestones in three Lanes, not a numeric track. A Lane can be completed once, so there is nothing to farm.
- **Agency, stated plainly:** with one defined reward per Lane, a player is not choosing a progression path. They are earning three predefined tools through different kinds of play. The choice is which Lane to chase and when. This is worth building only if the rewards create enough depth, which Stage 1 tests.

## Point 2. Lanes, and why they are not a checklist

**Settled: three Lanes, each with 2 or 3 qualifying feats, any one of which completes it.** A visible checklist turns the Arena into chores, and hidden feats can't be planned around. Lanes are broad styles of play, and the completion banner names what earned it. The exact feats are discoverable, not listed.

**Rule on scarcity.** The first rule was one non-scarce feat per Lane. It became **no consumable gating**: a feat shouldn't need a one-of-a-kind object that another player can use up, but a Lane doesn't have to be guaranteed. Explore and Endure may be unavailable late or in some lobbies, and that is intentional and measured.

### Explore (Reserve, Stages 2 and 3)

- Kill a Riftling whose Home is on a Shard other than the player's start Shard.
- Contribute about 25% of a Warden's maximum Health as effective damage during **one Engagement**.
- **Counters clear** when the Warden's regeneration starts, on a full reset, or on despawn. The Warden's lifecycle (hold, walk home vulnerable, heal after a delay) means counters must not clear during the walk home, when a third party can still finish it. Hit-and-run across regeneration cycles therefore can't accumulate.
- Credit comes the moment the threshold is reached. Last Hit still owns the Warden's loot.
- **A Far Shard feat was dropped.** Via the Crown every non-neighbour is about a 12 s trip, so it added almost nothing a Riftling kill didn't require, and a dwell timer would be artificial.

### Contest (Riposte, Stage 1)

Any one of:
- About **60 effective damage to players, summed across opponents** (after Shield, no overkill, accepted hits on living Match participants; no PvE, Collapse or self-damage).
- Eliminating a player.
- Triggering a Reaction on another player (credited to whoever triggered it).

Thresholds were compared at 40, 60 and 100. 40 is about one combo, so it is tagging. 100 duplicates a kill, and shared damage rarely reaches it. Summing across opponents counts shared and third-party fights, and a per-opponent rule would make 2-player lobbies impossible.

### Endure (Catch Breath, Stage 1 for Close Call, Stage 2 for Pressured Exit)

- **Close Call:** after a qualifying hit, Health is about 30 or below, and the player is still alive 10 s after the last qualifying damage (the timer resets on each).
- **Pressured Exit:** at least about 15 effective damage in the 10 s before a Section's removal, taken while standing on that Section during its Warning, and the player is alive on remaining ground after the settle window.
- **Only damage from players and Wardens qualifies.** Riftlings and Sentries do not. They are predictable, so a player could manufacture danger from them alone (a Riftling claw during a Warning, or tanking Sentry shots).

**Why it is shaped this way.** Brink Exit (leaving doomed ground in the last 5 s) was rejected because it rewards deliberately delaying an escape. A Match-clock Beat (alive at the first removal) rewards passive survival. Evade- and Shield-based feats were rejected because they are Kit-dependent or collude trivially. A raised threshold only slows a cooperating pair, and a source filter removes the solo version of the exploit.

**Accepted:** two consenting players can trade damage to complete Contest and Endure in about 25 s. In a one-life free-for-all only a feat that needs someone's death resists this, and that feat would end 2-player Matches and lock out PvE players. It is recorded as a known risk and is **not** solved by collusion detection.

## Point 3. Activation and pacing

**Settled: a Lane completes immediately, with a banner, and its reward becomes usable when both hold: 60 s have passed since GO, and the player has reached an out-of-combat moment after earning it.**

- **Why not immediate use:** Contest completes at 60 damage, usually while the player is already ahead in a fight, so an immediate Riposte would be an in-fight power spike for the winner.
- **The calm requirement applies once**, to the unlock. It does not revoke access in combat afterwards, or Riposte could never be used.
- **Why a 60 s gate:** it prevents an early Explore from giving an advantage before most Kits exist, and it doesn't depend on Kit state or Pickup luck.
- Rewards were once going to be banked as choice tokens and redeemed through a menu. That was dropped with the choice structure (see Point 5).

## Point 4. The combat-idle signal

**Settled: one server-authoritative signal, `combatIdleSeconds` of about 3, shared by Catch Breath, Reserve and the unlock rule.** Damage timestamps alone aren't enough: a player Evading or being attacked without damage must not count as calm.

It is stamped by: an accepted hit dealt or taken (including fully absorbed hits), a hit voided by Evade or Parry (**both sides**), the player's own damaging Cast, Basic swing or Charge start, PvE damage dealt or taken, and a Riposte activation. A rejected or invalid request never stamps. **No proximity-based state.** A missed projectile that passes you isn't detected unless it hits, which is accepted.

Two other "idle" notions already exist: the 7 s `fightIdleSeconds` used for fight logging, and the 7 s Natural Recovery delay. They are different things and aren't reused.

## Point 5. Rewards

**The philosophy test.** A reward should add one verb or one decision, be visible to opponents, have limited uses and be counterable, and not simply raise damage, Health, movement speed or Evade uptime. It should be orthogonal to the Ability roster and Dash. Real tactical advantage is allowed, and it need not be power-neutral. **Rewards are unlocked, not chosen:** one defined reward per Lane.

### How the structure changed

Binary choices (two rewards per Lane) were designed in full. Only about three of six candidates were compelling, and the second options were modest (Echo, Backtrace, Heavy Hand). A choice between a strong tool and a modest passive isn't a choice, and a shared pool fails arithmetically (three strong rewards, three picks). The token bank, the redemption gate, the Tool slot, the bench and the selector were removed.

### Contest: Riposte

A stationary frontal stance. A Sector hit from a player (Basic Attack, Arc Burst) that lands inside the window is voided like an Evade hit, and the defender's **Basic cooldown refreshes**, with the **next Basic within 1 s reaching about 9.5 studs instead of 8**. No damage bonus, stagger or forced movement.

- **Numbers (all tunable):** about 0.1 s startup, about 0.35 s window, about 0.4 s recovery on failure (Dash stays available), **3 fixed uses with no refunds** (a refund would let a skilled player parry a Basic-only attacker indefinitely).
- **Variants compared:** negate only (redundant with Evade), negate plus a 0.5 s stagger (too strong, and a control loss that is worst under latency), and negate plus a counterattack without a disable (chosen). 11 studs is a tuning ceiling, not a starting value, because it turns most retreats into hits.
- **Not parried:** projectiles, and PvE attacks. That keeps it out of Tickets 23 and 24.
- **Matchup dependence is accepted and measured.** It is strong against Basic and Arc Burst, and does nothing against Tidal Bolt or Rift Hook. Against those the answer is Evade.
- **Overlap with Evade:** distinct by being stationary, frontal, Sector-only, limited to 3 uses, and with a failure recovery. Not proven, so Evade overlap is watched.
- **Rift Lunge was dropped**, because Basic is already usable during a Dash and Blink Strike is a planned Ability. The principle: rewards are orthogonal to the Ability roster and to Dash.

### Endure: Catch Breath

A stationary channel of about 1.5 s that restores about 30 Health **on completion only**. Interrupted by movement, a Dash, an offensive action or any accepted hit (including a fully absorbed one). **An interrupted attempt consumes the use**, and a rejected activation consumes nothing. 2 uses. Refused in combat (the shared signal) and while Charging.

- **Why on completion, not gradual:** interrupting it must have a consequence. The checkpoint variant (half at the midpoint) is recorded for use if late interruptions prove too frustrating.
- **Value:** with the 3 s gate and Natural Recovery starting only after 7 s, it fills the gap after a fight. It is largely unusable inside a Last Stand fight.
- **Steady Nerves** (recovery starts sooner) was rejected as a permanent regeneration advantage, **Grounded and Grounded-lite** (knockback resistance) because they conflict with Stone Guard's "no Knockback immunity", with "Tidal and Hook keep their meaning" and with the Dais, which names Tidal Bolt the main ring-out threat.

### Explore: Reserve (Stage 3, gated)

One extra Ability held in storage, **not castable**. The full rules are in the ticket. In summary:

- It fills **only when a Pickup replaces a slot out of combat**: the displaced Ability goes into an empty Reserve instead of dropping. In combat, replacement behaves exactly as today and the Reserve is **inert**.
- It is swapped with Q or E by a **Reserve modifier + Q or E**, out of combat only. F + Q or E stays exclusively for Pickup replacement. The modifier key is undecided and is settled in the input review.
- The incoming Ability gets the normal 0.85 s Equip Lock. Cooldowns belong to the Ability and keep running while it is stored (an existing invariant).
- Ownership spans Q, E and Reserve, with one copy of each Ability (a separate `owns` lookup, with `find` unchanged so Reserve can never be cast). A duplicate Pickup upgrades whichever copy you own, in or out of combat.
- Elimination hands over Reserve last, atomically, inside `Kits.takeAll`.
- **Rejected:** auto-collecting into Reserve, a discard action, a dedicated swap selector, a fixed partner slot, and a one-time replacement buffer that can't swap back.

**Honest assessment, and why it is gated.** Reserve is the most invasive reward (Kit, Pickups, elimination, client copies, HUD, the 427-spec base). Its value is assumed, not measured. It is a half-step toward a third Ability slot, which the docs defer until about 6 to 8 Abilities exist, and it would confound that test. It also changes loot dynamics and Rank 2 frequency, which muddies the telemetry for the other two. So it is fully specified but built only on evidence, and **its decision is independent of whether Riposte and Catch Breath succeed**. If it proves valuable on its own, it should be reconsidered as a base Kit-system change.

### Fallback candidates (recorded, not built)

Echo (passive Cache-break and Elimination markers), Scout's Sweep (a delayed bucketed census, dropped as too weak after the safety limits), Survey (object outlines), Cache Cracker, Rift Seals (Spoils Ring, Hush, Anchor Ring), Feint, Heavy Hand, Read the Kit, Wager, Exposed Opener (dropped: a passive damage bonus that breaks the 40% single-hit cap with a Reaction), Backtrace (a bearing to the attacker), Danger Sense, and checkpoint healing. A second reward per Lane can be added later, on evidence.

## Point 6. Interactions and accepted risks

- **Stacking:** Reserve works only in calm, Riposte only in a fight, and Catch Breath is rarely usable inside the Last Stand. A rough upper bound is about 50 to 90 Health-equivalent for a flawless player against a melee or Arc opponent. That is an estimate, not data. Hoarding all 5 uses for the final is the real stacking case, and it is a tension, not a flaw.
- **Small lobbies:** Explore is nearly free in 2-player Matches (four Shards' Riftlings are untouched), and Endure often won't complete in short Matches. There is no lobby-size scaling. Completion is measured by lobby size.
- **Accepted and recorded:** Explore may be too easy in 2-player Matches, or unavailable too quickly in full lobbies. Reserve may give an early advantage to a PvE-first player. Players may feel forced to pursue Explore before the PvE actors disappear. Riposte may be too valuable against Basic and Arc and too weak against projectile Kits. Total defensive value (three Ripostes plus two Catch Breaths) is large even though it can't be used at once. Cooperating players can stage the damage feats, and Natural Recovery makes that nearly free.

## Point 7. Evidence, tuning and telemetry

Tuning is **problem-specific**, not a strict order:

| If | Then |
|---|---|
| Too much total Riposte value | Reduce the uses |
| Parrying is too easy | Shorten the window |
| The reply is too reliable | Reduce its reach |
| Catch Breath restores too much | Reduce the healing |
| Catch Breath enables repeated resets | Reconsider the uses or the activation conditions |
| Reserve enables excessive Kit cycling | Raise the combat-idle requirement or the Equip Lock |
| Friend pairs unlock everything early | Raise the Contest threshold or the Close Call Health requirement |
| Endure is solved by PvE | Exclude PvE damage from the remaining routes |

**No Dais-specific exclusions or extra combat restrictions without evidence.** Catch Breath's power is not balanced around cooperative trading. It is judged on legitimate Matches first.

**Telemetry.** Rewards earned and usable-at times by lobby size, with earned-but-never-activated distinguished from activated-but-never-used. For Riposte, activations, successes, failures and **prevented damage** (measured, not just counted), by opponent Kit. For Catch Breath, completions, interruptions and actual Health restored. For Endure, the **source of each completion**. A flag for two players' Contest completions within about 5 s of each other from mutual damage with no Elimination. Time-to-all-three, and the rewards held when entering the Dais. For Reserve's decision, demand signals from Stage 1: replacements, Abilities dropped by replacement, and Pickups left in the world at the Match end.

## What was dropped, and why

Echo as Explore's only reward (too subtle to carry a Lane), the Tool slot with a bench (only two actives, in opposite states, so two dedicated actions suffice), the Match-clock Beat as a fallback (passive survival), Evade-based feats (collusion is free), and per-lobby scaling (no evidence).
