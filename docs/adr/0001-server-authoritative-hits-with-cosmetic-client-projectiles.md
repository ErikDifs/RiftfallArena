# Server-authoritative hits with cosmetic client projectiles

For the first combat prototype, casts must feel instant but hits must be fair, so the caster's client immediately plays the animation, sound and a Cosmetic Projectile while the server alone simulates the Authoritative Projectile and decides all hits and damage. A Cast and its Aim Point are sent once, and other clients rebuild their Cosmetic Projectile from that event, so projectile positions are never replicated per frame.

A Chargeable Ability may send a `ChargeStart` message before its Cast, as set out in [ADR-0006](0006-server-timed-charging-with-a-two-stage-cast.md). The Cast and its Aim Point are still sent once, on release, Charge state isn't streamed, and server-authoritative hit resolution is unchanged.

## Rules

- Damage, status effects, knockback, hit markers and meaningful impact effects happen only after Server Confirmation.
- A Cosmetic Projectile may predict environment impacts, but must not show a convincing player or NPC hit before Server Confirmation.
- A Rejected Cast cancels the local visuals and restores the local cooldown state.
- Hit Tolerance stays small and intentional. It covers capsule size and simulation differences, and does not broadly turn misses into hits. Projectile radius is separate, and an attack's forgiveness comes from its own radius, range or width.

## Considered Options

- **Server only:** rejected because waiting for the server makes every cast feel delayed.
- **Client claims hits, server checks them:** rejected because it is easier to exploit and lets victims be hit after reaching cover.

## Consequences

- Target rewind and lag compensation are deliberately left out. Add them only if playtests show that server-authoritative hits feel unfair.
- Players will occasionally see near misses that disagree with their screen, which is the accepted cost of this model.
- The first ability set has no long-range hitscan or sniper-style abilities. It favours visible-travel-time projectiles, AoE, cones and short-range abilities.
