# Riftfall Arena

A fast Roblox multiplayer action game that mixes PvP and PvE in short matches. This glossary is the shared language for its design and code.

## Language

### Products

**Riftfall**:
The universe and brand that the games belong to.
_Avoid_: Using "Riftfall" alone to mean the game

**Riftfall Arena**:
The game being built now, and the proving ground for the universe's combat.
_Avoid_: Plunderstorm clone, the MMO

**Riftfall Realm**:
A possible future persistent-world RPG. It is not in scope and should not influence Arena's design.
_Avoid_: Riftfall MMO

### Match

**Match**:
One free-for-all round of about 12 minutes, moving through Gather, Clash and the Rift Collapse. The last surviving player wins.
_Avoid_: Game, round, session

**Gather**:
The opening phase, where players explore, fight monsters and assemble a Build, with quick respawns.
_Avoid_: Early game, farming phase

**Clash**:
The middle phase, where stronger rewards pull players into contested areas and respawns get slower.
_Avoid_: Mid game

**Rift Collapse**:
The final, time-driven phase, where unstable Rifts consume the playable area and death is permanent.
_Avoid_: Storm, shrinking zone, circle

### Builds

**Kit**:
Everything a player can use at a given moment: their Basic Attack, Dash and Ability Slots.
_Avoid_: Loadout, class

**Basic Attack**:
The untyped, non-Reactive, short-range melee attack every player always has, which cannot be replaced.
_Avoid_: Auto attack, primary

**Ability**:
A combat action found during a match that occupies an Ability Slot.
_Avoid_: Spell, skill, power

**Ability Slot**:
One of three places in a Kit that can hold any Ability.
_Avoid_: Hotbar slot, spell slot

**Pickup**:
An Ability of a specific Rank lying in the world that a player can collect. A full Kit needs a deliberate hold-to-replace, which drops the replaced Ability as a new Pickup at its current Rank.
_Avoid_: Drop, loot item

**Rank**:
An owned Ability's or Pickup's power level, from 1 to 3, which also serves as its rarity. Rank 2 improves numbers, and Rank 3 also adds one behaviour change.
_Avoid_: Rarity tier, level, Common/Rare/Epic

**Upgrade**:
An increase in an owned Ability's Rank, gained by collecting a Pickup of that Ability. It does not use another Ability Slot.
_Avoid_: Level up, stack

**Build**:
The combination of Abilities and Upgrades a player has assembled during the current match.
_Avoid_: Spec, loadout

### Combat

**Crosshair**:
The fixed mark at the centre of the screen in the over-the-shoulder camera, which shows where the player is aiming.
_Avoid_: Cursor, reticle

**Aim Point**:
The spot in the world under the Crosshair, where aimed and ground-targeted abilities are directed.
_Avoid_: Mouse position, target

**Natural Recovery**:
The health a character regains after a period without taking damage, which lets disengaging reset a fight.
_Avoid_: Regen, heal

**Kill Recovery**:
The partial health restored to a player for killing another player.
_Avoid_: Lifesteal, kill heal

**Hitbox**:
The simple, fixed volume the server uses as a character's target for all combat, independent of its visible model. All players share the same dimensions.
_Avoid_: Collision box, body parts, avatar size

**Prototype Character**:
The single, uniform appearance every player uses in the first combat prototype.
_Avoid_: Default avatar, test rig

**Hit Tolerance**:
The small, deliberate margin that covers projectile radius, character capsule size and simulation differences. It never changes an attack's path or turns clear misses into hits.
_Avoid_: Aim assist, soft lock, lag compensation

**Cast**:
A player's single use of an Ability or Basic Attack, sent once with its Aim Point. The server either accepts or rejects it.
_Avoid_: Shot, fire event

**Charging**:
The exclusive state of holding an Ability's input to build a stronger Cast, with slower walking and a Telegraph. Its Cast happens on release.
_Avoid_: Channelling, wind-up, Dash charges

**Authoritative Projectile**:
The server's simulation of a projectile, and the only version that can hit anything.
_Avoid_: Real projectile, server bullet

**Cosmetic Projectile**:
A client-drawn projectile rebuilt from a Cast, purely visual. It can predict environment impacts but never confirms a hit on a player or NPC.
_Avoid_: Fake projectile, predicted hit

**Server Confirmation**:
The server's verdict on a hit. Damage, status effects, knockback, hit markers and meaningful impact effects all wait for it.
_Avoid_: Hit event, client hit

**Rejected Cast**:
A Cast the server refuses. The caster's client cancels the local visuals and restores its cooldown display.
_Avoid_: Failed cast, cancelled cast

### Monsters and world rewards

**Monster Family**:
A themed group of monsters with one defining behaviour, recognizable territory and a shared Drop Table, such as Tide creatures.
_Avoid_: Monster type, faction

**Elite**:
A stronger version of a Monster Family's monster, with one additional Telegraphed attack or behaviour.
_Avoid_: Boss, mini-boss

**Telegraph**:
The readable wind-up in the world before a meaningful attack lands.
_Avoid_: Cast bar, warning

**Threat**:
A simple configurable value that lets damage make a monster switch target.
_Avoid_: Aggro

**Home Territory**:
The area a monster or spawn group belongs to, and returns to if pulled too far.
_Avoid_: Leash area, camp

**Last Hit**:
The final damaging hit on a monster, which decides its authoritative killer.
_Avoid_: Kill steal, tag

**Drop Table**:
The configurable, weighted list of Pickups and Ranks a reward source can produce.
_Avoid_: Loot table, drop pool

**Rift Cache**:
A world object along a useful route that gives at least a Rank 1 Pickup. Anyone can take or contest it.
_Avoid_: Chest, catch-up reward

### Interactions

**Status**:
A temporary, server-owned condition on a character that can feed a Reaction, such as Wet or Frozen. It must be readable in the world, not only in the UI.
_Avoid_: Buff, debuff, effect

**Reaction**:
A shared, data-defined outcome that occurs when a typed effect meets a matching Status. Its definition decides what happens to that Status and who it can target.
_Avoid_: Combo, synergy

**Applied By**:
The character that put a Status on its target.
_Avoid_: Owner, source

**Triggered By**:
The character whose hit caused a Reaction, and who currently receives its immediate damage and kill credit.
_Avoid_: Owner, killer

**Wet**:
The Status applied by Water, which sets up the stronger Lightning Reaction.

**Frozen**:
The Status a target reaches after enough Frost, which sets up Shatter. It is separate from Stun.
_Avoid_: Stun, Ice

**Frost Immunity**:
A short period after Frozen ends during which a target cannot gain Frost, so that it can't be chain-frozen.
_Avoid_: Freeze protection, diminishing returns

**Shatter**:
The Reaction caused by a Shatter-capable hit on a Frozen target.
_Avoid_: Break, freeze burst

**World Zone**:
A future area of the map that holds a Status-like condition, such as Oil on the ground, and feeds the same Reaction system. It is not in the prototype.
_Avoid_: Surface, puddle, hazard

### Movement

**Dash**:
A short burst of movement in the player's movement direction, limited by charges and cooldown. It starts instantly on the owning client, and the server can reject it.
_Avoid_: Dodge, roll

**Evade**:
A short server-owned window at the start of an accepted Dash during which incoming hits have no effect. It does not remove existing Statuses.
_Avoid_: I-frames, invulnerability, dodge

**Knockback**:
A forced push whose direction and strength are decided by the server.
_Avoid_: Fling, displacement

**Root**:
A server-owned state that prevents a character from moving.
_Avoid_: Snare, immobilize

**Stun**:
A server-owned state that prevents a character from moving or acting.
_Avoid_: Disable, incapacitate

**Movement Validation**:
The server's periodic check that a player's movement is explained by legitimate sources, such as normal speed, Dashes and Knockback.
_Avoid_: Anti-cheat, speed check

**Last Valid Position**:
The most recent position the server accepted for a player, and where it returns them after clear or repeated violations.
_Avoid_: Checkpoint, rollback point
