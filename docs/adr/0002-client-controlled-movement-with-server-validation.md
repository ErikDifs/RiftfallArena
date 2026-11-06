# Client-controlled movement with server validation

Movement is the game's second-highest priority and must feel immediate, so for the first prototype the owning client moves its own character and starts Dashes instantly, while the server decides what is allowed and checks the result. This uses standard Roblox characters and avoids the cost of a custom controller while we are still proving the combat is fun.

## Rules

- The server is authoritative over whether a Dash is allowed, including charges, cooldown, state restrictions, and Root and Stun conditions. It can reject a Dash the client has already started, and the client then reconciles.
- Movement Validation runs at a low fixed rate rather than every frame. It accounts for known legitimate movement: normal speed, Dash displacement, Knockback, Rift Step, and future movement buffs and debuffs.
- The server keeps a Last Valid Position and corrects a player only on clear or repeated violations, so that one lag spike does not rubber-band a normal player.
- For player Knockback, the server decides whether it happens and its direction and strength, then the owning client applies the impulse. The server allows for that impulse in its validation window, and a client that clearly ignores or contradicts it is treated as suspicious.
- For NPC Knockback, the server owns and applies it directly.
- The server owns Root and Stun state. The client respects it locally for responsiveness, and the server rejects illegal abilities and actions even if a modified client keeps moving.
- Projectile and combat hit decisions use the server's known positions, with no lag compensation, as set out in [ADR-0001](0001-server-authoritative-hits-with-cosmetic-client-projectiles.md).

## Considered Options

- **Server moves the character:** rejected because every Dash would be delayed by the player's ping.
- **Custom predicted character controller:** the strongest model, but rejected for now because it replaces Roblox character movement and is too large for a solo prototype.

## Consequences

**Accepted weakness:** this model reduces obvious movement cheating, but cannot fully prevent a modified client from making small illegal movements or ignoring some physical effects. Revisit this before public launch if playtesting or exploit testing shows it is a real problem.
