# Match progression through one-time Milestones

Decided 2026-11-10, while settling [Ticket 25](../milestone-2-tickets.md#25-match-milestones). The reasoning and every rejected option are in [`milestone-design.md`](../milestone-design.md).

During a Match, a player can earn up to three **Milestones**, one per **Lane**: Explore, Contest and Endure. Each Lane is completed once, by a qualifying feat that comes out of ordinary play, and grants one defined tactical reward: **Reserve** (Explore), **Riposte** (Contest) and **Catch Breath** (Endure). A reward becomes usable after a global time gate and the player's first out-of-combat moment. Everything is Match-local. Nothing persists, there is no currency, and there are no choice menus.

Milestones are **not XP, levels or a power curve**. A Milestone is not an Ability Rank, and no reward changes an Ability's Rank.

## Why

Ability Ranks are the only Ability upgrade system. On the old Match map the run test measured a first Ability in about 7.8 s and a full Kit in about 17 s (floor pacing, perfect route knowledge), so a Kit stops changing early. Milestones give a player something to pursue and decide about after that point, without a second stat track. The rewards are small tactical tools with limited uses, visible counterplay and no raw increase to damage, Health or speed.

## Rejected

- **XP and levels.** A numeric curve needs a source table, and every source can be farmed. Kill XP feeds the leader in a one-life, six-player game, and PvE XP rewards farming. It would also be a second power system beside Rank, with a glossary term ("level") the project already avoids.
- **A token bank with a binary choice per Lane.** It was designed in full, then dropped when only about three of the six candidate rewards were compelling. A choice between a strong tool and a modest passive isn't a real choice. It also cost a redemption gate, a selector and a bench.
- **A shared reward pool.** With three Lanes and three strong rewards, every player ends up with all of them, so the pool is a pick order, not a choice.
- **Doing nothing but logging.** Kept as the fallback: if Stage 1's run tests show Milestones don't change play, the system is reconsidered before it grows.

## Consequences

- A reward must add a verb or a decision, be visible to opponents, have limited uses and be counterable. Raw stat rewards are out.
- The system is staged so each part is built only on evidence: core Milestones first, Arena and PvE integration after Tickets 21 to 24, and Reserve only if run tests justify its Kit and Pickup changes.
- Cooperating players can stage the damage feats. This is accepted for the prototype, not solved, and is tuned through the rewards' power, not through anti-collusion machinery.
- If Reserve proves valuable independently, it should be revisited as a base Kit-system decision (the two-slots versus three question), not kept as a reward.
