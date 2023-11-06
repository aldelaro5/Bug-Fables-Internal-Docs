# Beemerang
Vi's Beemerang when thrown.

## Data Arrays
None, but `vectordata[0]` is used as an internal field.

## Remarks
The only entity that exists of this type is the one that exists under the `Ressources/prefabs/objects/Beerang` prefab as it comes with its own NPCControl of this `objecttype`. There is no way under normal gameplay other than PlayerControl requesting it to have it instantiated meaning effectively, if an entity has this `objecttype`, it is the same than if it was the player `beemerang` because it cannot be of any other type and no other NPCControl of this type can exist.

## Special behaviors
- This type features an enabled `ccol` unlike other objects
- This type doesn't doesn't feature an `scol` and the field remains null (The SphereCollider isn't added)
- This type cannot be added to the player.`npc` list on LateUpdate
- This type doesn't have its `ccol` height and center adjusted in LateUpdate according to `colliderheight`

## Setup
First, the entity's `sound` is set to loop and the `RangeHold` sound is eet to play at half volume.

Then, all collisions between any collider whose GameObject's tag is `Respawn` and the `scol` or the entity's `ccol` are ignored.

Finally, all collisions between the colliders in the map's `entityonly` and the `scol` or the entity's `ccol` are ignored. Nothing happens if this array was null or empty.

## Update
First, the tag is set to `BeeRang`.

Then, if the entity's `sound` isn't playing, call [PlaySound](../../EntityControl/EntityControl%20Methods.md#PlaySound) on it with the `RangHold` clip set to loop,

If `hit` is false, the following section occurs.

### `hit` false logic
If the distance between this object and `vectordata[0]` is higher than 0.2, this object's position is set to a lerp from the existing one to `vectordata[0]` with a factor of the game's frametime * the entity's `speed`.

Otherwise, if the ability key is held, [flag](../../../Flags%20arrays/flags.md) 21 is true (got Beemerang Halt), `heldonce` is false and `WackaWorm.disablehold` is false:
- The entity's `sound` pitch is set to 1.25
- The entity's `spin` is set to (0.0, 0.0, 30.0)
- `timer` is set to 99.0
- If `particles` is null, it's initialised to an instance of `Prefabs/Particles/ContinuousSmokeCloud` childed to this object at this object's position + Vector3.up * 0.2 with angles (-90.0, 0.0, 0.0)

Otherwise, if the `timer` is exactly 99.0:
- `particles` is put offscreen at (0.0, -9999.0, 0.0) then destroyed in one second and set to null
- The entity's `sound` pitch is set to 1.25
- `heldonce` is set to true
- The entity's `speed` is set to 0.15
- `vectordata[0]` is set to the entity's `startpos` + the normalised direction from the entity's `startpos` to this object * 5.0. This essentially places this object 5.0 away from the `startpos` in the same direction is was already away from it.
- `timer` is set to -1.0

Otherwise, `hit` is set to true.

---

No matter what, if the `timer` is higher than -2.0, it is decremented by framestep.

If the distance between this object position and the player is higher than 0.45 the following section occurs (otherwise, this object is destroyed).

### Too far logic
If the `timer` is less than -2.0, the ability key is held, [flag](../../../Flags%20arrays/flags.md) 21 is true (got Beemerang Halt), `heldonce` is false and `WackaWorm.disablehold` is false, the following happens:
- The entity's `sound` pitch is set to 1.25
- The entity's `spin` is set to (0.0, 0.0, 30.0)
- `timer` is set to -50.0
- If `particles` is null, it's initialised to an instance of `Prefabs/Particles/ContinuousSmokeCloud` childed to this object at this object's position + Vector3.up * 0.2 with angles (-90.0, 0.0, 0.0)

Otherwise, if the ability key isn't held or the `timer` is exactly -50.0:
- `particles` is put offscreen at (0.0, -9999.0, 0.0) then destroyed in one second and set to null
- `heldonce` is set to true
- `timer` is set to -100.0
- This object's position is set to a lerp from the existing one to the player position with a factor of the game's frametime * 0.3

## OnTriggerEnter if the other gameObject layer is `Ground`, `NoDigGround` or `BeerangOnly`
If `hit` is false, the `WoodHit` sound is played at 1.2 pitch and 0.5 volume and the HitPart particles are played at this object position + Vector3.Up.

`hit` is then set to true and `timer` to -3.0.
