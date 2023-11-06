# SetPlayerRespawn
A trigger zone that sets the player's spawn point ???

## Data Arrays
- `vectordata[0]`: The position to set the player `lastpos` when entered

## Setup
If `vectordata[0]`'s magnitude is 0.1 or higher, the `boxcol`'s layer is set to 0 (default) which ends this object's setup.

Otherwise, `boxcol` is recreated as a new GameObject named `respawner` with a new trigger BoxCollider that is childed to the map. This implies all fields relating to the `boxcol` are overriden. This collider's absolute position is set to the entity's `startpos`, the angles to the transform's angles, the tag to `Respawn` and the isStatic to true. Finally, the gameObject gets destroyed ???

## Update
If the `timer` hasn't expired yet, it is decremented by the game's frametime clamped from 0.0 to infinity. Otherwise, if it is 0.0 and the entity isn't `dead`, a [Death](../../EntityControl/Notable%20methods/Death.md) coroutine is started with the entity.

## OnTriggerStay
If the magnitude of `vectordata[0]` is above 0.1 and the other gameObject is the player, the player `lastpos` is set to `vectordata[0]`.
