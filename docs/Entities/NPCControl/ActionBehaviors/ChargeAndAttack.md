# ChargeAndAttack
The same than [ChasePlayer](ChasePlayer.md), but also attack the player depending on the [AnimID](../../../Enums%20and%20IDs/AnimIDs.md) when getting closer than a configurable threshold.

## Frequency meaning
The minimum distance to the player at which no attacks will occur. If the distance falls below this number, a ChargeAndAttack coroutine will be started. NOTE: if it is above 10.0 or negative, the value is overriden to 2.0.

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

From there, if the distance between this and the player is less than `frequency` (after overriding it as described above), `behaviorroutine` is set to a ChargeAndAttack coroutine that is started.

## ChargeAndAttack
TODO