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
One free-for-all round of about 12 minutes, moving through Gather, Clash and the Rift Collapse, in which each player has one life. The last surviving player wins.
_Avoid_: Game, round, session

**Eliminated**:
The state of a player whose one life in a Match has ended. They never respawn into that Match, and can only Spectate or Return to Hub.
_Avoid_: Dead, knocked out, out

**Spectate**:
Watching the living players of a Match after being Eliminated, with no effect on play.
_Avoid_: Ghost, observer mode

**Hub**:
The place outside any Match where players gather, respawn freely and enter Matches, and where Return to Hub takes an Eliminated player.
_Avoid_: Lobby, menu

**Gather**:
The opening, lower-pressure phase, where players explore, fight monsters and assemble a Build.
_Avoid_: Early game, farming phase

**Clash**:
The middle phase, where stronger rewards pull players into contested areas.
_Avoid_: Mid game

**Rift Collapse**:
The final, time-driven phase, where unstable Rifts consume the playable area so nobody can hide.
_Avoid_: Storm, shrinking zone, circle

### Builds

**Kit**:
Everything a player can use at a given moment: their Basic Attack, Dash and Ability Slots.
_Avoid_: Loadout, class

**Starting Kit**:
The Kit a player has each time they enter play: either the Reference Kit, or an empty one that the run fills.
_Avoid_: Default loadout, spawn kit

**Reference Kit**:
The fixed Kit of Tidal Bolt and Arc Burst used to validate the core combat, including in Stage 3. It stays available for testing after Kits start being found during a run.
_Avoid_: Default class, starter build

**Basic Attack**:
The untyped, non-Reactive, short-range melee attack every player always has, which cannot be replaced.
_Avoid_: Auto attack, primary

**Ability**:
A combat action found during a match that occupies an Ability Slot.
_Avoid_: Spell, skill, power

**Ability Slot**:
One of the places in a Kit that can hold any Ability. Milestone 2 has two, on Q and E. The long-term number is unresolved, with three, on Q, E and R, kept as a likely test.
_Avoid_: Hotbar slot, spell slot

**Pickup**:
An Ability of a specific Rank lying in the world that a player can collect. A Pickup that fits, into a free slot or as an Upgrade, is collected just by coming near it. With a full Kit, the player has to choose a slot to replace, and the replaced Ability drops as a new Pickup at its current Rank.
_Avoid_: Drop, loot item

**Equip Lock**:
The short time after an Ability enters a slot, by replacement or by being collected into a free slot, during which it can't be Cast. An Upgrade doesn't cause one.
_Avoid_: Swap cooldown, equip delay

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
The small, deliberate margin that covers character capsule size and simulation differences. It never grows to make an attack more forgiving, never changes an attack's path and never turns clear misses into hits.
_Avoid_: Aim assist, soft lock, lag compensation

**Sector**:
A flat, horizontal slice of a circle in front of a Cast's origin, set by its range, width and a height allowance above and below. It follows only the horizontal part of the aim, so camera pitch never tilts it. Arc Burst and the melee Basic Attack hit with one.
_Avoid_: Cone, arc, 3D cone

**Cast**:
A player's single use of an Ability or Basic Attack, sent once with its Aim Point. For a Chargeable Ability, that is on release. The server either accepts or rejects it.
_Avoid_: Shot, fire event

**Casting Style**:
How an Ability's input becomes a Cast: Instant, Committed Cast or Chargeable.
_Avoid_: Cast type

**Instant**:
A Casting Style where pressing the input Casts immediately.
_Avoid_: Reaction, for a quick defensive response

**Committed Cast**:
A Casting Style with a fixed, Telegraphed wind-up the player doesn't control, which may commit them depending on the Ability.
_Avoid_: Charged cast, channel

**Chargeable**:
A Casting Style where holding the input is Charging, and releasing it Casts.
_Avoid_: Hold-to-fire, which is how the Basic Attack repeats

**Charging**:
The exclusive state of holding a Chargeable Ability's input to build a stronger Cast, with slower walking and a Telegraph. Its Cast happens on release.
_Avoid_: Channelling, wind-up, Dash charges

**Tap Window**:
The short time after pressing a Chargeable Ability's input within which a release is an exact tap, with no Charge, slowdown or Telegraph.
_Avoid_: Dead zone, minimum Charge

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
A breakable world object along a useful route that drops at least a Rank 1 Pickup when broken. Anyone can break it, and anyone can take or contest what it drops.
_Avoid_: Chest, catch-up reward

**Sentry**:
A stationary hostile PvE enemy that fires at nearby players and guards valuable Rift Caches. It is the only hostile PvE until monsters are built.
_Avoid_: Turret, tower, monster

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
A forced displacement, away from or toward its source, whose direction and strength are decided by the server. Rift Hook's pull is Knockback.
_Avoid_: Fling, pull state

**Shield**:
Temporary protection that absorbs damage before health for a short time. A Shielded hit still lands, with its Statuses, Knockback and Reactions, which separates it from Evade.
_Avoid_: Armour, barrier, absorb

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
