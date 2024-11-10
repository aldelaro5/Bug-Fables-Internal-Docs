# Respawn
[PlayerControl](../PlayerControl/PlayerControl.md) contains a field that sets the position to bring the player to when they should be respawned: `lastpos`.

This field is automatically set during [TransferMap](Map%20loading.md#transfermap) and [MoveInside](Insides.md#moveinside), but it's also possible to manually set it as well as have 2 ways to manage it from within the map:

1. Define a [SetPlayerRespawn](../Entities/NPCControl/ObjectTypes/SetPlayerRespawn.md) whose `vectordata[0]` corresponds to the position to set `lastpos` when the player collides with its `boxcol` (If `vectordata[0]` magnitude isn't higher than 0.1, it acts like the second method described below since the `Respawn` tag is applied to the `boxcol` automatically)
2. Define any trigger collider in the map whose GameObject has the `Respawn` tag. In this case, if the player collides with the collider while `onground` for more than 15.0 frames (tracked via the player's `respawncount` field), the player's `lastpos` is set to its current position

Essentially, either define an NPCControl with the position to set `lastpos`, or define a collider with a tag that resets `lastpos` to the player current position periodically. Defining the NPCControl without a position to set will act as a plain `Respawn` tagged collider.
