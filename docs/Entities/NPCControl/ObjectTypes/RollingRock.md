# RollingRock
A rolling rock that either appears and rolls on its own or is shot from a canon. The target direction, velocity, radius, rotation, spawn timing and minimum y position limit are all configurable with the option to only have the rock roll once it hits the ground and an option to only activate the rock when another NPCControl `hit` value is true.

## Data Arrays
- `data[0]`: If 1, the rock starts rolling when it hits the ground with some particles, sound effect and a shake screen. If 0, the rock always rolls when the `actioncooldown` expires. Any other value is considered invalid and will cause illogical behaviors
- `data[2]`: 1 if a canon should shoot the rock, no canon is setup otherwise
- `data[3]`<sup>1</sup>: The NPCControl at the map entity id whose `hit` value determines if the rock is considered active. A positive number means to test for it being true while a negative one test for it being false using the absolute value as the map entity id. This is optional, the rolling rocks remains active if it doesn't exist or is -1
- `vectordata[0]`: The target vector of the rock which determines towards where it will start rolling and its x/z velocity
- `vectordata[1].x`: The minimum y position allowed of the rock before respawning it
- `vectordata[1].y`: The radius of the rock. Defaults to 3.0 if it was less than 0.1
- `vectordata[1].z`: The `actioncooldown` of the canon which is the time before shooting if applicable or the time it takes to respawn it if we aren't shooting from a canon
- `vectordata[2]`: The rotation angles to apply each update where the rock rolls

1: NOTE: a caveat with this value is it is only possible to test for the value being false on the id 0, it is NOT possible to test it being true. Also, it is NOT possible to test for id 1 being false because -1 is a special value to opt out of this feature and that meaning takes priority

## Setup
- entity.`onground` is set to false
- entity.`alwaysactive` is set to true
- The entity.`ccol` is enabled overriding the default [Start](../Start.md) logic for objects
- If `vectordata[1].y` is less than 0.1, it is set to 3.0
- The `scol` center is set to (0.0, `vectordata[1].y`, 0.0)
- The `model`'s local scale is set to Vector3.one * `vectordata[1].y`
- The `model`'s local position is set to (0.0, `vectordata[1].y`, 0.0)
- The `scol` radius is set to `vectordata[1].y`
- The `jumpheight` is halved
- `internaldata` is initialised to a new array with 2 elements being 0.0 and 100.0
- If `data[2]` is 1, see the section below about setting up shooting from a canon
- All collision between the entity.`ccol` and the player.entity.`ccol` are ignored
- The ground detector is recreated completely via [CreateFeet](../../EntityControl/EntityControl%20Methods.md#CreateFeet)

### When we are shooting from a canon (`data[2]` is 1)
- `actioncooldown` is set to `vectordata[1].z`
- `internaltransform` is initialised to have 2 transforms:
  - The first is an instance of the `Prefabs/Objects/Cannon` prefab childed to the current map with a scale of Vector3.one * (`vectordata[1].y` / 2.0), a position of the entity.`startpos` and with angles incremented by (0f, 0f, -90f).
  - The second is the first child from the root of the canon instance (the cylinder nozzle)
- The canon is set to rotate its y angle (not its x and z) to face the canon's current position + `vectordata[0]`. This aligns it horizontally to its target
- The entity.`rigid` is locked via [LockRigid(true)](../../EntityControl/EntityControl%20Methods.md#lockrigid)
- The position of the the rolling rock is set to (0.0, 999.0, 0.0) which is offscreen in the air

## Update
First, if `data[2]` is present and 1 (we are shooting from a canon), a complete logic is performed to scale `internaltransform[1]` (the nozzle of the canon) at different points in the shooting and to update the `actioncooldown`. No matter if this applies or not, the main logic section can apply after.

### Canon nozzle scaling
This section involves 2 countdown timer: `internaldata[0]` and `actioncooldown`. The former is reset to `internaldata[1]` (100.0) after shooting and represents the interval before preparing to shoot again while the latter represents the interval before respawning the rock. They both interact with each other, but this section still remains independant of what happens after when applicable.

On the first update, `internaldata[0]` is expired, but not `actioncooldown` which will cause this object to be continuously positioned offscreen (0.0, 999.0, 0.0). If this happens while the `data[3]` `hit` check condition is fufilled or not applicable, this is where the `actioncooldown` is decremented by the game's frametime. On top of this, if less than 60.0 frames is left on the `actioncooldown`, the local scale of the nozzle is set to a lerp from the existing one to (0.5, 1.25, 1.25) with a factor of the 1/20 of the game's frametime. This scale signals that the canon is about to shoot by warping it mostly from the side for a brief moment.

Eventually, the `actioncooldown` expires and this is detected by going below 0.0, but remaining above -1000.0. When this happens:
- The entity.`rigid` gets its velocity zeroed out
- This object's position is set to the entity.`startpos` + the normalized version of `vectordata[0]` * `vectordata[1].y` + Vector3.up * 0.25 (this places the rock back to its original position)
- [LockRigid(false)](../../EntityControl/EntityControl%20Methods.md#LockRigid) is called on the entity to unlock its `rigid`
- The `actioncooldown` is set to -1100.0 which prevents it from entering this code block until the next shooting
- `internaldata[0]` is set to `internaldata[1]` (100.0)
- The nozzle of the canon has its local scale set to (1.25, 0.75, 0.75) which warps the canon on the opposite direction than when set earlier the instant the shooting happens

As a result of this, `actioncooldown` is frozen below -1000.0, but this causes on each further updates to have the local scale of the canon nozzle to be set to a lerp between the existing one and Vector3.one with a factor of a 1/20 of the game's frametime.

From there `internaldata[0]` hasn't expired which causes further updates to decrement it by the game's frametime. Eventually, it will expire which means we are back to where we started on the first update and the cycle repeats itself.

Overall, it means that:
- Every time `actioncooldown` has 60.0 or less left, the scale of the nozzle is warped to be a bit smaller horizontally
- When `actioncooldown` expires, the shooting logic happens with a position set and a scale of the nozzle set to be larger horizontally
- The nozzle scale slowly gets back to normal
- After 100.0 frames passed, the logic resets back to where it started with a fresh `actioncooldown`

This repeates over and over until the `data[3]` check is violated which freezes the `actioncooldown` and prevents this section from doing any logic until the check is fufilled again.

### Main logic
After the section above if applicable, a check is done if the rock should have its `hit` set to true. It is set to true if any of the following conditions is true:
- We aren't shooting from a canon and `data[0]` is 0 or the entity is `onground`
- We are shooting from a canon and `actioncooldown` expired.

If we are setting `hit` to true, a special case happens when if it was false before and `data[0]` is 1. In that case, the `impactsmoke` particle is played at this object position and if the distance between the player and this object is less than 15.0, the `Thud` sound is played followed by a ShakeScreen using the vector (0.1, 0.35) without reset.

#### `hit` value handling
From there, what happens depends on the value of `hit`.

If it is true:
- The x and z components of the entity.`rigid` velocity are set to `vectordata[0].x` and `vectordata[0].z` respectively
- The entity.`model` angles are incremented by `vectordata[2]` * framestep
- If `data[3]` is present and the NPCControl at the map entity id of its value has its `hit` set to true, then [PlaySound](../../EntityControl/EntityControl%20Methods.md#PlaySound) is called with the `RollingRock` clip looped. If `data[3]` is present, but the `hit` check is violated, the entity.`sound` is stopped. NOTE: see the note on the data arrays section above for a caveat with this

If `hit` is false, the entity.`sound` is stopped as long as it is playing, it exists and the clip is still `RollingRock`. The entity.`sound` is then set to no longer loop.

#### Common steps
Regardless of everything, if this object's y position is below `vectordata[1].x` WarpRock is called (see the section below for details).

## OnTriggerStay
This does nothing if we are in a `pause`, `minipause` or `inevent` and that if a battle is in progress that it's not `inevent`.

If the other gameObject is the player and it's not `digging`, the [event](../../../Enums%20and%20IDs/Events.md) 116 (Getting crushed by a `RollingRock`) is called with this as the caller.

Otherwise, if the other gameObject tag is `RockLimit`, [BreakRock](../BreakRock.md) is called.

## OnCollisionEnter
If the other gameObject is an `Object` NPCControl of type [BreakableRock](BreakableRock.md), [BreakRock](BreakableRock.md#breakrock) is called on the other's NPCControl.

## BreakRock
This is a public method that has logic specific to this object type.

- If the player is less than 15.0 away from this object and the `RockBreak` sound wasn't playing, it is played at 0.5 volume
- entity.`sound` is stopped
- If the player is less than 15.0 away from the rolling rock, a ShakeScreen happen with 0.1 as the amount for 0.35 seconds without reset
- A `Prefabs/Objects/CrackRockBreak` prefab instantiated at the rolling rock position with its CrackRockBreak's initialcolor set to the rolling rock material color and a scale of Vector3.one * half of `vectordata[1].y`
- entity.`rigid` has its velocity zeroed out
- entity.`onground` is set to false
- If `data[2]` is 1 (we are shooting from a canon) the following occurs (if not, WarpRock is called instead, see the section below for details):
  - The rolling rock position is offscreen in the air at (0.0, 999.0, 0.0)
  - The entity.`rigid` is locked via [LockRigid(true)](../../EntityControl/EntityControl%20Methods.md#lockrigid)
  - `actioncooldown` is reset to `vectordata[1].z`

## WarpRock
This is a private method specific to this object type that prepares to respawn the rock.

If we are shooting from a canon (`data[2]` is 1), [LockRigid(true)](../../EntityControl/EntityControl%20Methods.md#LockRigid) is called on the entity and the `actioncooldown` is reset to `vectordata[1].z`.

If we aren't shooting from a canon:
- This object's transform is set to (0.0, 999.0, 0.0) which is offscreen in the air
- LatePos is called which sets the position of the entity to its `startpos` in `vectordata[1].z` seconds.
- `hit` is set to false
- The entity.`rigid` velocity is zeroed out
- The entity.`onground` is set to false

All of this prepares the next rock to spawn.

## Hazards.OnObjectStay
If the NPCControl passed has this type, it returns true which allows it to stay in collision with the Hazards.