# DigWall
A DigWall is a GameObject with a `DigWall` tag that contains a collider. All these colliders are stored in `digwall` on GetDigWall which is only invoked by Start. If the GameObject contains multiple collider, only the first one will be stored. It is intended for the GameObject to only feature a single collider.

These colliders have a special property: on every LateUpdate, their enablement are always updated depending on if the `player` is `digging` or not. They are enabled only if the `player` isn't digging and disabled otherwise.

The intent is to have colliders that can act as a wall, but disappear once digging which effectively allows the player to go "through" a wall physically, but visually look as if they are digging under it. Since the `ccol` is still present when `digging`, this is how maps can have walls that looks like they can be passed under, but in reality, these are completely blocking walls whose colliders disables when digging.
