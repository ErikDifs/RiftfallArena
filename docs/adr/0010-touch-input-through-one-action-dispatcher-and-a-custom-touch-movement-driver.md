# Touch input through one action dispatcher and a custom touch movement driver

Decided 2026-11-12, while settling [Ticket 29](../milestone-2-tickets.md#29-platform-input-and-mobiletablet-support). Phones and tablets are to be fully playable, with no unfair advantage over PC and no change to server authority.

Touch is a **second input source** feeding the same device-independent action dispatcher as keyboard and mouse. `Input` keeps its callbacks, so Casting, Dash, Replacing and Spectating don't learn which device a player uses. On touch devices Riftfall **replaces Roblox's native touch movement**: `DevTouchMovementMode = Scriptable`, a small touch-only driver with a floating left stick and our own Jump button, calling `Humanoid:Move(worldDirection, false)` from the camera's yaw. PC and gamepad movement stay Roblox-native. Every touch is owned by its `InputObject` from the moment it begins, and the touch router, not Roblox's GUI layer, decides what each touch does. Server authority, remotes and payloads are unchanged: the server never learns the device.

## Why

The over-the-shoulder camera is `Scriptable` and the character is forced to the camera's yaw every frame. Roblox's native touch controls own the stick's touch zone and the Jump button's position and size, and neither can be changed without forking the player scripts. The design depends on facts those controls would decide for us: which touches rotate the camera, where Jump sits relative to Dash, and that auto-jump is off. One owner for every touch is the only way to make "a drag from a button keeps rotating the camera", "a touch's role never changes", and "never more than two required touches" true by construction.

## Considered options

- **Roblox's native touch controls, customized.** Least code, but the stick zone and Jump button belong to Roblox. Touches can leak into the camera, and Jump can't sit beside Dash. No supported way exists to move only the Jump button.
- **Forking the player scripts.** Gives control of the layout, at the price of thousands of lines of Roblox code that drift with every engine update. Rejected as the maintenance cost the custom driver avoids.
- **`ContextActionService` or Roblox's newer action system for everything.** Native, and gamepad would come almost free. But it gives no control over touch ownership, dragging from a button into the camera, or cancellation, and rewriting `Input` around it puts PC behaviour at risk. The newer system is still changing (the player scripts carry feature flags for it).
- **A separate touch layer calling Casting and the others directly.** Duplicates the Match gate, the Spectating override and Ticket 27's read-only hook, so touch and PC behaviour would drift apart.

## Consequences

- PC behaviour must not change. Characterization specs of today's keyboard and mouse semantics come first, then a **minimal** extraction (holder, edge and cancel state only). A full rewrite of `Input` into a bindings-driven abstraction was rejected as more risk than the feature needs.
- Holders are counted **per source, one per action**, so a lost touch-end can never hold an action for ever. A cancelled Charge never casts: touch cancellation maps to a per-slot cancel, not to every cancel path. A real focus loss keeps its existing behaviour.
- The custom driver is ours to maintain: dead zone, trailing origin, Jump, safe-area placement and cancellation near lethal edges. Auto-jump is off on touch for parity.
- Gamepad support is a later source plus native movement, with no change to the dispatcher. It is a separate ticket.
- Known competitive asymmetries are **logged and measured, not compensated** (see Ticket 29): the Fire to Dash thumb lift, no Jump during a Charge, the replace hold, continuous stick angles and touch sensitivity. If device evidence shows one is unacceptable, the escalation is a recorded decision (a Fire latch, a shoulder Jump or a Charge latch), never a silent rule change.
- Several Roblox behaviours this relies on are unverified and are explicit implementation checks in Ticket 29. A failed check changes the plan, not the principle.
