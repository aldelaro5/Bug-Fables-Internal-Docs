# Event
Start the [event](../../../Enums%20and%20IDs/Events.md) whose id is `eventid` with this NPC as the caller.

## args meaning
None.

## Interact
Start the [event](../../../Enums%20and%20IDs/Events.md) whose id is `eventid` with this NPC as the caller.

## PlayerControl.LateUpdate
entity.`emoticonid` gets set to 1 (a blue ?) and entity.`emoticoncooldown` gets set to 2.0 if all of the following conditions are met:
- We aren't in a `pause` or `minipause`
- The [message](../../../SetText/Notable%20states.md#message) lock is released
- The player isn't `digging`, `flying`, `startdig` or in `shield`
- This is the closest NPC in the `npc` list as of the last 3 frames