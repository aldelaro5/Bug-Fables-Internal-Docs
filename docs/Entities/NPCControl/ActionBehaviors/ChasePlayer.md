# ChasePlayer
Approach the player as close as possible within the `radiuslimit`.

## Frequency meaning
None ???

## DoBehavior
The following occurs on the entity:
- If the NPCControl `returntoheight` is true, `height` is set to a lerp from the existing one to `initialheight` with a factor of 0.1
- The `sprite` is enabled
- `forcemove` is set to false
- [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) is called with the player position
- `emoticonid` is set to 2 (!) and `emoticoncooldown` is set to 2.0
- If [HasGroundAhead](../../EntityControl/EntityControl%20Methods.md#hasgroundahead) returns true, [Move](../../EntityControl/Notable%20methods/Move.md) is called to the player with the `speedmultiplier` speed and the `Chase` [animstate](../../EntityControl/Animations/animstate.md). Otherwise, [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called
- `oldstate` is set to -1
- `oldfly` is set to false
- The `animstate` is set to `Chase`
- The position is limited to a radius of `radiuslimit` where the center is `startpos` ignoring the y component

## Update (Common, end)
If this is the current behavior and we are in `minipause` or `inevent`, then [StopForceMove](../EntityControl/EntityControl%20Methods.md#StopForceMove) is called on the entity.
