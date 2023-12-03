# UpdateCollider
This update methods is called by [LateUpdate](Unity%20events/LateUpdate.md). It only applies for `npcdata` being present and every 2 frames, but because [LateUpdate](Unity%20events/LateUpdate.md) only calls it every 3 frames when applicable, it's actually only going to process every 6 frames when `npcdata` is present.

First, if the entity `iskill` is false and the `npcdata`'s `colliderheight` is 0.0, the height and radius of the `ccol` goes back to their `initialcolliderdata` and `colliderheight` is initialised to the `initialcolliderdata`.x which is the initial height of the `ccol`.

From there, the method only proceeds if the player is free or that the `npcdata`'s `insideid` matches the current one while the entity isn't `dead`, `iskill` or in the process of dying (`deathcoroutine` not being null). The cycle is done if it isn't. NOTE: technically, `lastpos` is set to the `transform`'s position if the player was free, but the condition failed one way or another, it's just that this doesn't change anything in practice because no one will read from it until the end of [LateUpdate](Unity%20events/LateUpdate.md).

Finally, the `transform`'s position is set to `lastpos` and if there was a `forcemove`, it is stopped.
