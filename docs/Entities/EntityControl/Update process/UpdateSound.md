# UpdateSound

This update method is called on [LateUpdate](Unity%20events/LateUpdate.md), but also on PlaySound. It handles adjusting the `sound`'s volume and panStereo. This method only applies when `sound` is present and for anything except `npcdata` of `MusicRange`.

First, the panStero is adjusted if the `sound` is playing to the `campos`'s x mapped lerped from -1.0 to 1.0 and then clamped to the proper side depending on the `transform`'s actual x position. For example, if the entity is positioned relatively to the left of the camera, the clamp will be from -1 to 0, but from 0 to 1 if it was positioned more to the right and from -1 to 1 if both x exactly match. This essentially helps to account both the viewport position and the actual position to determine where to pan the `sound`.

Finally, the volume is adjusted. When unpaused, the new `sound`'s volume is adjusted to `soundvolume` * the game's soundvolume or the pausemenu's svolume if the game is paused at the Settings or Key Bindings window.
