# ChasePlayer
Approach the player as close as possible within the `radiuslimit` or until there is no more ground to move in the direction of the player.

## Frequency meaning
None, overriden to 2.0 if it's not between 0.0 and 10.0 inclusive.

## Additionaldata
- `radiuslimit`: The max radius in which the entity is allowed to move in.

## DoBehavior
- If `returntoheight` is true, entity.`height` is set to a lerp from the existing one to entity.`initialheight` with a factor of 0.1
- entity.`sprite` is enabled
- entity.`forcemove` is set to false
- [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) is called on the entity with the player position as the facing point
- entity.`emoticonid` is set to 2 (!) and entity.`emoticoncooldown` is set to 2.0
- If [HasGroundAhead](../../EntityControl/EntityControl%20Methods.md#hasgroundahead) returns true on the entity, [Move](../../EntityControl/Notable%20methods/Move.md) is called on the entity to move it to the player position with the `speedmultiplier` speed and the `Chase` [animstate](../../EntityControl/Animations/animstate.md). Otherwise, [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called
- entity.`oldstate` and entity.`oldfly` is set to -1 (this forces a sprite update)
- The [animstate](../../EntityControl/Animations/animstate.md) is set to `Chase`
- The position is limited to a radius of `radiuslimit` where the center is `startpos` ignoring the y component

## Update (Common end logic)
If this is the current behavior (according to `inrange`) and we are in `minipause` or `inevent`, then [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity.
