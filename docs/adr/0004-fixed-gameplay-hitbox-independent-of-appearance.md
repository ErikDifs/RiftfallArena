# Fixed gameplay Hitbox, independent of appearance

Roblox avatars vary widely in size and shape, so hits based on body parts would give small avatars a real advantage. All combat therefore tests against a simple, fixed Hitbox, which the server uses for projectiles, cones, close-range attacks, AoE overlap checks, Status application and Knockback eligibility. Every player has the same Hitbox dimensions, and each monster type has a configured Hitbox sized for fair gameplay rather than derived from its rendered mesh. An avatar must never provide a gameplay advantage through size.

## Considered Options

- **Own avatars with hits based on body parts:** the Roblox default, rejected as unfair.
- **Own avatars with a fixed Hitbox:** the expected direction before public release, but deferred so that the first playtests judge the combat rather than avatar compatibility.
- **One uniform Prototype Character with a fixed Hitbox:** chosen for the first combat prototype.

## Consequences

Adding avatars later should not change combat rules, because hits never depended on body parts. That work must still constrain or scale avatars to roughly fit the gameplay silhouette, test unusual proportions and accessories, make sure animations work, keep Status visuals readable, and consider hiding or restricting accessories that obscure combat information.
