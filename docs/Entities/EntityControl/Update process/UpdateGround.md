# UpdateGround

This update method is called by [LateUpdate](Unity%20events/LateUpdate.md). It updates the `ccol`'s physics material frictions and the `jumpcooldown`.

The `ccol` physics material update applies except if it's a `npcdata` of `Pushrock`. The staticFriction and dynamicFriction of the material are either set to 1.0 or 0.0 depending on specific conditions:

* If `onground`, the `deltavelocity`'s magnitude is lower than 0.2 and `jumpcooldown` is expired
  * Then 1.0 is set to the friction unless the `transform` is the player's and the [animstate](../Animations/animstate.md) isn't `Walk` and `hitwall` is false in which case, 0.0 is set to the friction values
* 0.0 is set to the friction values otherwise.

Finally, if the `jumpcooldown` is still active, it is decreased by framestep.
