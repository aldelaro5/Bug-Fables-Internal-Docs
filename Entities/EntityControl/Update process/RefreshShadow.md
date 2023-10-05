# RefreshShadow

RefreshShadow is a method on EntityControl called by [LateUpdate](Unity%20events/LateUpdate.md) when applicable to refresh the rendering of the `shadow` object which is a gradient black circle. Since all of this is rendering, only a high level explanation is needed.

The method will skip the update if the `shadow` isn't present, but also under a specific set of conditions that have to be true:

* The entity has an `npcdata` and its `freezecooldown` and `timer` expired
* The entity is not an `item`
* The entity is `onground`
* It is not a `battle` entity
* The squared distance with `lastshadow` is extremely small (\<= 0.005)

If all those conditions are true, updates are skipped every other frames. This is on top of [LateUpdate](Unity%20events/LateUpdate.md) precondition checks before this method is even called.

On an update cycle, shadows remain enabled as long as `iskill` is false and that either the `sprite` / `model` is present or that `overrideshadow` is true. If none of those conditions are true, `shadow` gets disabled instead of getting updates. 

On an actual update, a raycast is done from slighly above the entity down. If it gets a hit withing 10.0 units, `shadow` remains enabled as long as the entity isn't `digging` with the position, scale and color updated to match the hit point, `ccol`, `height`, `shadowsize` and the `sprite`'s alpha. More specifically

* Position is slightly above the hit point (this prevents z fighting)
* the scale is `shadowsize` 
* the result of `ccol`.radius * (1.0 - Mathf.Abs(`hitInfo`.point.y - (`transform`.position.y + `height`))) / 10.0 clamped from 0.0 to infinite all normalised for the x and y (the z is locked to 1.0) This essentially means the `shadow` scale is caped at `shadowsize` * `ccol`'s radius and gets scaled down the further away the hit point is from the `height` up to 10.0 units away at which points, it gets scaled to zero.
* The angles are set to look at the normal of the hit point which will make the shadow angle match the point it hit on the ground
* The color is the sprite's alpha clamped to 0.4

If the raycast didn't hit, `shadow` is disabled. If the raycast is hit, but the entity was `digging`, then it still gets updated, but disabled.
