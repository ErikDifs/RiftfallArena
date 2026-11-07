# Server-timed Charging with a two-stage Cast

A Chargeable Ability needs a stronger Cast the longer its input is held, and that strength decides geometry, damage and Knockback, so a client must not be able to choose it. The server therefore derives Charge only from when it receives the two messages of a Chargeable Cast, and clients never send a Charge percentage or duration. This amends the single-message Cast in [ADR-0001](0001-server-authoritative-hits-with-cosmetic-client-projectiles.md) for Chargeable Abilities only: Instant Abilities and the melee Basic Attack keep the normal `CastRequest` path.

## Rules

- **Start:** on press, the client sends `ChargeStart` with the Ability and an attempt identifier, and no Charge, duration or Aim Point. The server validates that Charging can begin, and records its own start time against that attempt. This is not yet a Cast.
- **Release:** on release, the client sends the attempt identifier and its current Aim Point, and this is the Cast request. The server verifies the matching active attempt, derives Charge from its own timestamps, applies the Tap Window, maximum Charge and clamping, validates the Aim Point and all normal Cast rules, and then performs the Cast.
- **Attempt identifier:** it only has to be unique within a player's own sequence of attempts. A release is valid only for the matching, currently active attempt, so a stale one can never release a newer Charge, start a cooldown or create a Cast.
- **Late release:** a release for an attempt the server has already interrupted, cancelled, expired or replaced is a Rejected Cast, with no Cast and no cooldown. The caster's local visuals cancel under the existing rules, and a Cast that briefly appears and disappears in this rare race is accepted.
- **Caster's visuals:** on release, the caster immediately draws the Cast from its own local estimate of Charge, without waiting for the server. A cosmetic projectile already fired isn't resized or corrected if the server's value differs. The authoritative geometry always uses the server's value. Revisit this only if real playtesting shows a noticeable mismatch.
- **Other clients:** `CastStarted` carries the server's Charge value, and they build their Cast visuals from it, not from their locally reconstructed Telegraph.
- **Telegraphs:** Charging is exposed through replicated state, and Charge is never streamed.
- **End cause:** the server owns the cause wherever it knows enough: accepted Dash, Knockback, Frozen, Stun, death, and timeout as `Expired`. Any `DashRequest` that arrives while Charging ends it as `Dash`, whether or not the Dash is then accepted, because the client has already cancelled on its side. A client abandoning Charging for any other reason is logged as `ClientCancelled`, without trusting a claimed cause. This protects the metrics and adds no gameplay rules.

## Considered Options

- **Client-supplied Charge percentage:** rejected because a modified client could simply claim full Charge.
- **Client-supplied Charge duration:** rejected for the same reason, with an extra step.
- **Continuous Charge percentage streaming:** rejected as needless traffic that the server still couldn't trust.
- **Periodic low-frequency Charge streaming:** rejected for the same reasons.
- **The server choosing an Aim Point at expiry:** rejected because the server may decide whether a Cast is valid, but must not invent where the player aimed.

## Consequences

Accepted security properties:

- A modified client can't grant itself extra Charge, because the server's clock and clamps decide it.
- Sending `ChargeStart` early gives no useful advantage, because the Charging costs and state begin correspondingly.
- Withholding a release ends in `Expired`, not indefinite Charge.
- Stale releases are rejected through the attempt identifier.
- A modified client can hide its own Telegraph locally, but other clients still receive the authoritative Charging state.
- A modified client may try to ignore the Charging slowdown locally, which remains subject to the accepted Movement Validation limits in [ADR-0002](0002-client-controlled-movement-with-server-validation.md).

A lag spike on `ChargeStart` alone makes the server's Charge shorter than the caster's estimate, so the real bolt can be smaller than the one drawn.
