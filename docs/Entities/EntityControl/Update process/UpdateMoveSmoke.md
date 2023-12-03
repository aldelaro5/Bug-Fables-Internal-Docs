# UpdateMoveSmoke
This update method is called both on [LateStart](../Notable%20methods/LateStart.md) and [LateUpdate](Unity%20events/LateUpdate.md). It handles the enabling of `movesmoke` and adjusting its local position. This method only applies if `movesmoke` is present.

First, we ensure the object is active if it wasn't already. Then, its local position is either set to zero or offscreen. The former happens if `overridemovesmoke` is false, the `height` is 0.0, the entity isn't `dead`, the [animstate](../Animations/animstate.md) is `Walk` or `Chase`, we are unpaused, the `npcdata` is either not present or its `freezecooldown` has expired, the `sprite`'s alpha is higher than 0.9 and we aren't `digging`.

Otherwise, it is moved offscreen.
