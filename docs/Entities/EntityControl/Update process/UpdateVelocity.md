# UpdateVelocity
This update method is called by [LateUpdate](Unity%20events/LateUpdate.md). It handles adjustements of the `rigid`'s velocity and the `startscale`.

First, we check if the velocity should be clamped. The condition to check is very similar to the one in [LateUpdate](Unity%20events/LateUpdate.md) which means most calls will end up performing this part, but there is one exception. While the method checks for the `npcdata`'s `istrapped` to be false which [LateUpdate](Unity%20events/LateUpdate.md) doesn't do, it only has any effect if none of the other prerequisite are true, but they are all present on [LateUpdate](Unity%20events/LateUpdate.md) except the check for a `battle` entity. This means if it's a `battle` entity, it may not execute if the `npcdata` has a `trapped` of false and has the type `NPC` on odd frame count. In practice, this doesn't happen because a `battle` entity wouldn't have an `npcdata` which always satisfy both checks. The frame count for both [LateUpdate](Unity%20events/LateUpdate.md) and this method are based on parity so regardless of what happens, this will execute every 2 frames.

The actual clamping can be done in 3 ways depending on the conditions (checked in order):

* It's an [item entity](../Item%20entity.md): The `rigidi`'s velocity is clamped from (-10.0, -20.0, -10.0) to (10.0, 10.0, 10.0)
* It's the player and they are dashing: Only the y velocity is clamped from -20.0 to `jumpheight` * 1.5
* If neither of the above are true, then the x and z are clamped from -10.0 to 10.0 and the y is clamped from -20.0 to infinity

Next, if `shrink` is true, then the `startscale` is lerped from itself to zero with a factor of framestep * 0.09

Finally, the y velocity is zeroed if `onground`, the existing y velocity is under -5.0 and either the `npcdata` is not present or it is with a `dizzytime` \<= 0.0
