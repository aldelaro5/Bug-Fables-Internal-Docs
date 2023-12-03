# Beemerang
Vi's Beemerang when thrown. This object type is only created dynamically.

## Data Arrays
- `vectordata[0]`: The desired position that the Beemerang should end up befor going to the opposite direction. This is set by PlayerControl.DoActionTap under normal gameplay.

## Remarks
The only entity that exists of this type is the one that exists under the `Ressources/prefabs/objects/Beerang` prefab as it comes with its own NPCControl of this `objecttype`. There is no way under normal gameplay other than PlayerControl requesting it to have it instantiated meaning effectively, if an entity has this `objecttype`, it is the same than if it was the player `beemerang` because it cannot be of any other type and no other NPCControl of this type can exist.

## Special behaviors
- This type features an enabled entity.`ccol` unlike other objects
- This type doesn't doesn't feature an `scol` and the field remains null (The SphereCollider isn't added)
- This type cannot be added to the player.`npc` list on LateUpdate
- This type doesn't have its entity.`ccol` height and center adjusted in LateUpdate according to `colliderheight`

## Setup
The entity.`sound` is set to loop and the `RangeHold` sound is eet to play at half volume.

All collisions between any collider whose GameObject's tag is `Respawn` and the `scol` or the entity.`ccol` are ignored. This is to avoid collisions with independant [SetPlayerRespawn](SetPlayerRespawn.md).

All collisions between the colliders in the map.`entityonly` and the `scol` or the entity.`ccol` are ignored. Nothing happens if this array was null or empty.

## Update
The tag is set to `BeeRang`.

if the entity.`sound` isn't playing, call [PlaySound](../../EntityControl/EntityControl%20Methods.md#PlaySound) on it with the `RangHold` clip set to loop,

The `hit` value tracks on which half of the path the beemerang is on. If it's false, it's on the first half and if it's true, it's on the way back. Either gives a different branch of logic.

### `hit` false logic
If the distance between this object and `vectordata[0]` is higher than 0.2, this object's position is set to a lerp from the existing one to `vectordata[0]` with a factor of the game's frametime * the entity's `speed`. This basically moves the beemerang to its destination.

Otherwise (meaning the Beemerang reached its destination), if the ability key is held, [flag](../../../Flags%20arrays/flags.md) 21 is true (got Beemerang Halt), `heldonce` is false and `WackaWorm.disablehold` is false:
- The entity.`sound` pitch is set to 1.25
- The entity.`spin` is set to (0.0, 0.0, 30.0)
- `timer` is set to 99.0 (which gives it access to a new destination on the first Update cycle when the ability button is released)
- If `particles` is null, it's initialised to an instance of `Prefabs/Particles/ContinuousSmokeCloud` childed to this object at this object's position + Vector3.up * 0.2 with angles (-90.0, 0.0, 0.0)

Otherwise, if the `timer` is exactly 99.0 (meaning we were using Beemerang Halt, but released the ability input):
- `particles` is put offscreen at (0.0, -9999.0, 0.0) then destroyed in one second and set to null
- The entity.`sound` pitch is set to 1.25
- `heldonce` is set to true
- The entity.`speed` is set to 0.15
- `vectordata[0]` is set to the entity.`startpos` + the normalised direction from the entity.`startpos` to this object * 5.0. This essentially sets the destination to this object 5.0 away from the entity.`startpos` in the same direction it was already away from it (so it overshoots entity.`startpos` by 5.0). The way back will not be performed until it has also reached this second destination
- `timer` is set to -1.0 (the default value in preparation of doing the back path)

Otherwise, `hit` is set to true.

### `hit` is true logic
If the `timer` is higher than -2.0, it is decremented by framestep.

If the distance between this object position and the player is higher than 0.45 the following section occurs (otherwise, this object is destroyed because it means the Beemerang completed its full travel).

If the `timer` is less than -2.0 (meaning the Beemerang collided with a wall), the ability key is held, [flag](../../../Flags%20arrays/flags.md) 21 is true (got Beemerang Halt), `heldonce` is false and `WackaWorm.disablehold` is false, the following happens (this whole logic allows the beemerang to stall in place even if its path got cut short by a collider):
- The entity.`sound` pitch is set to 1.25
- The entity.`spin` is set to (0.0, 0.0, 30.0)
- `timer` is set to -50.0
- If `particles` is null, it's initialised to an instance of `Prefabs/Particles/ContinuousSmokeCloud` childed to this object at this object's position + Vector3.up * 0.2 with angles (-90.0, 0.0, 0.0)

Otherwise, if the ability key isn't held or the `timer` is exactly -50.0 (meaning the beemerang is ready to go back to the player):
- `particles` is put offscreen at (0.0, -9999.0, 0.0) then destroyed in one second and set to null
- `heldonce` is set to true
- `timer` is set to -100.0
- This object's position is set to a lerp from the existing one to the player position with a factor of the game's frametime * 0.3

## OnTriggerEnter if the other gameObject layer is `Ground`, `NoDigGround` or `BeerangOnly`
If `hit` is false, the `WoodHit` sound is played at 1.2 pitch and 0.5 volume and the HitPart particles are played at this object position + Vector3.Up.

`hit` is then set to true and `timer` to -3.0. This forces the beemerang to stop its path and to be on the way back. The only way to stall it before actually coming back is by using Beemerang Halt because the `timer` value allows it to stall until the ability button is released.

## Hazards.OnObjectStay
If the NPCControl passed has this type, it returns true which allows it to stay in collision with the Hazards.