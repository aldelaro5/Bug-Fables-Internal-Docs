# Update
This is the Update of EntityControl. It doesn't do as much as [LateUpdate](LateUpdate.md)

* Check if the entity should receive updates which only happens when unpaused, minipause isn't active, the [message](../../../../SetText/Notable%20states.md#message) lock is released and the entity isn't dead. This can be overriden with `activeonpause`
  * Follow the standard animation speed update logic unless `overrideanimspeed` is true which will set the animation speed to 1.0 if there's no `icecube` and to 0.0 if there is one on top of forcing the [animstate](../../Animations/animstate.md) to `Hurt`
  * Make sure the `sprite` is disabled if this is the player entity and is a submarine, otherwise
    * Toggle the sprite being enabled if the `icooldown` hasn't expired yet and the player is free (meaning it is present, no minipause or pause, not in an event, the [message](../../../../SetText/Notable%20states.md#message) lock is released, and the player isn't digging)
    * Ensure the sprite is enabled if it's not an item and it has no `npcdata` or it has one and it doesn't have a `disguiseobj`
    * Ensure the child of the sprite are enabled or disabled according to the main sprite's enabled unless the `npcdata` is present and its `trapped` is true
  * Decrease the `icooldown` with the framestep
  * Handle the restore of the entity when unpausing happend:
    * The `rigid`'s useGravity is set to `lastgravity`
    * The `sound`'s volume is set to `lastvolume`
    * The `rigid`'s velocity is set to `lastvelocity`
    * `lastvelocity` and `pausepos` are put back to null
* Handle pausing updates, if we are paused
  * Unless `lastvelocity` already had a value (which means this is the first frame of being paused) the pause states are saved
    * `pausepos` is set to the `transform`'s position
    * `lastgravity` is set to the `rigid`'s useGravity
    * `lastvolume` is set to `sound`'s volume
    * `lastvelocity` is set to the `rigid`'s velocity
    * gravity is disabled on the `rigid`
    * The `sound` is muted
    * The `rigid`'s velocity is zeroed
  * If not in a battle, ensure the animation speed is set to 0.0 and then call [StopMoving](../../EntityControl%20Methods.md#StopMoving) with the current [animstate](../../Animations/animstate.md) which will zero the `rigid`'s velocity and the `deltavelocity`. Otherwise, ensure the animation speed is 1.0
