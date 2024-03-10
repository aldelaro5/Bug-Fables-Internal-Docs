# SetPlayerRespawn
A trigger zone that sets the player.`lastpos` which is the player last spawn point. It can either be managed by this NPCControl or be a completely independant collider that PlayerControl can collider with.

## Data Arrays
- `vectordata[0]`: The position to set the player `lastpos` when entered. If the magnitude is less than 0.1, the whole collider is managed indepedently from any NPCControl

## Setup
There are 2 modes this object can operate on:

- `vectordata[0]`.magnitude is 0.1 or higher: this NPCControl manages the `lastpos` and the colliders to it via its OnTriggerStay
- `vectordata[0]`.magnitude is less than 0.1: this object is destroyed after creating an independant GameObject with a BoxCollider that PlayerControl's OnTriggerStay interacts with

In the case of NPCControl management, the `boxcol`'s layer is set to 0 (default) which ends this object's setup.

In the case of independant management by recreating `boxcol` a new GameObject named `Respawner` with a new trigger BoxCollider that is childed to the map with the following properties: 

- position is set to the entity.`startpos`
- angles is set to this transform's angles
- tag is set to `Respawn` (which allows it to be detected by PlayerControl and others) 
- isStatic is set to true

Finally, the gameObject gets destroyed. From now on, this NPCControl cease to exist because the entire logic is managed by an independant GameObject. The tag gives it special behaviors:

- The [Beemerang](Beemerang.md) entity.`ccol` and `scol` have their collisions ignored with the collider
- OnTriggerStay of PlayerControl: if the other collider has the tag with a `vectordata[0]` above 0.1, `lastpos` is set to it.
- OnTriggerExit of PlayerControl: `respawncount` is set to 0.0 which resets a failsafe to prevent too much infinite respawns.
- OnTriggerStay of RayDetector: prevents entity.`hitwall` to change when colliding with an object with this tag.

This means the Update and OnTriggerStay of this document only applies when the collider is managed by this NPCControl.

## Update (When managed by this NPCControl)
If the `timer` hasn't expired yet, it is decremented by the game's frametime clamped from 0.0 to infinity. Otherwise, if it is 0.0 and the entity isn't `dead`, a [Death](../../EntityControl/Notable%20methods/Death.md) coroutine is started with the entity.

## OnTriggerStay (When managed by this NPCControl)
If the magnitude of `vectordata[0]` is above 0.1 and the other gameObject is the player, the player `lastpos` is set to `vectordata[0]`.
