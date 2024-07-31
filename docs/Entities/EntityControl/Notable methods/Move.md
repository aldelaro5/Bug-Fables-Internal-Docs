# Move
This method is called by [FixedUpdate](../Update%20process/Unity%20events/FixedUpdate.md) during a `forcemove`, but it can be called externally. It will set the velocity of the movement and handle other logic for this instant. It receives the target position, a multiplier to the velocity and the [animstate](../Animations/animstate.md) to use during the movement.

First, the entity is unifixed if `fixedentity` is true with the `rigid` constraints being set to freeze only rotations. Unless `overrideanim` is true, the [animstate](../Animations/animstate.md) is set to the state sent in.

This is where the facing logic occurs which involves having `moverotater` look at the target position sent in and call [FaceTowards](../EntityControl%20Methods.md#facetowards) with the same position (this sets `flip` and `backsprite` accordingly).

From there, the `rigid`'s velocity is set to a vector where each component is towards the target (uses `moverotater.forward.normalized` which was aligned earlier) * `speed` * the multiplier sent in. However, the y component is exempted from this if `ignorey` or the `rigid` has its gravity enabled. In such case, the y component is the existing one.

Then, the special case where the `walktype` is Jump is handled here. What happens is if the entity is `onground` and the `jumpcooldown` expired, it will also jump on top of that. This is further enhanced by playing the sound `AhoneynationHopJump` for a `Ahoneynation` [AnimID](../../../Enums%20and%20IDs/AnimIDs.md) and by playing `Jump` at 0.85 pitch for a `JumpingSpider`.

Finally, the `deltavelocity` is updated to the horizontal direction towards the target * `speed` * the multiplier sent in.
