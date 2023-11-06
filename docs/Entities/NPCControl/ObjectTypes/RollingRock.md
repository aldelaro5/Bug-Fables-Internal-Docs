# RollingRock
A rolling rock that either appears and rolls on its own or is shot from a canon.

## Data Arrays

- `vectordata[0]`: The offset vector of the rock (also determines the canon's facing direction if applicable)
- `vectordata[1].x`: The minimum y position allowed of the rock before respawning it
- `vectordata[1].y`: The radius of the rock. Defaults to 3.0 if it was less than 0.1
- `vectordata[1].z`: The `actioncooldown` of the canon which is the time before shooting if applicable or the time it takes to respawn it if we aren't shooting from a canon
- `vectordata[2]`: The rotation angles to apply each update where the rock rolls
- `data[0]`: ??? (determines some stuff to make it roll)
- `data[1]`: ???
- `data[2]`: 1 if a canon should shoot the rock, no canon is setup otherwise
- `data[3]`: The NPCControl at the map entity id whose `hit` value determines if the rock is considered active. A positive number means to test for it being true while a negative one test for it being false using the absolute value as the map entity id. This is optional (NOTE: a caveat with this is it is only possible to test for the value being false on the id 0, it is NOT possible to test it being true. Also, it is NOT possible to test for id 1 being false because -1 is a special value to opt out of this feature and that meaning takes priority)

## Setup
A few adjustements on the entity is performed:
- `alwaysactive` is set to true
- The `ccol` is enabled overriding the default [Start](../Start.md) logic for objects, but the collision between it and the player's `ccol` are ignored
- The `jumpheight` is halved
- The `model`'s local scale is set to Vector3.one * `vectordata[1].y`
- The `model`'s local position is set to (0.0, `vectordata[1].y`, 0.0)
- The ground detector is recreated completely via [CreateFeet](../../EntityControl/EntityControl%20Methods.md#CreateFeet) and the `onground` set to false (this happens at the end of SetUp even if we are initialising a canon)

Additionally, the `scol` is ensured to be enabled and trigger. Its center is set to (0.0, `vectordata[1].y`, 0.0) and its radius to `vectordata[1].y`. Then, `internaldata` is set to {0.0, 100.0}.

From there, `data[2]` is checked to see if we this rock should be shot from a canon. If we are, the following occurs on top:
- `actioncooldown` is set to `vectordata[1].z`
- `internaltransform` is initialised to have 2 transforms:
  - The first is an instance of the `Prefabs/Objects/Cannon` prefab childed to the current map with a scale of Vector3.one * (`vectordata[1].y` / 2.0), a position of the entity's `startpos` and with angles of (0f, 0f, -90f).
  - The second is the first child from the root of the canon instance (the cylinder nozzle)
- [FaceTowards](../../EntityControl/EntityControl%20Methods.md#FaceTowards) is called to face the canon's current position + `vectordata[0]`
- The entity's `rigid` is locked via LockRigid(true)
- The absolute position of the root transform is set to (0.0, 999.0, 0.0) which is offscreen in the air

## Update
First, if `data[2]` is present and 1 (we are shooting from a canon), a complete logic is performed to scale `internaltransform[1]` (the nozzle of the canon) at different points in the shooting and to update the `actioncooldown`. No matter if this applies or not, the main logic section occurs.

### Canon nozzle scaling
This section involves 2 countdown timer: `internaldata[0]` and `actioncooldown`. The former is reset to `internaldata[1]` (100.0) after shooting and represents the interval before preparing to shoot again while the latter represents the interval before respawning the rock. They both interact each other, but this section still remains independant of what happens after when applicable.

On the first update, `internaldata[0]` is expired, but not `actioncooldown` which will cause this object to be continuously positioned offscreen (0.0, 999.0, 0.0). If this happens while the `data[3]` `hit` check condition is fufilled or not applicable, this is where the `actioncooldown` is decremented by the game's frametime. On top of this, if less than 60.0 frames is left on the `actioncooldown`, the local scale of the nozzle is set to a lerp from the existing one to (0.5, 1.25, 1.25) with a factor of the 1/20 of the game's frametime. This scale signals that the canon is about to shoot by warping it mostly horizontally for a brief moment.

Eventually, the `actioncooldown` expires and this is detected by going below 0.0, but remaining above -1000.0. When this happens:
- The entity's `rigid` gets its velocity zeroed out
- This object's position is set to the entity's `startpos` + the normalized version of `vectordata[0]` * `vectordata[1].y` + Vector3.up * 0.25 (this places the rock back to its original position)
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
After the section above if applicable, a check is done if the rock should have its `hit` set to true. If we aren't shooting from a canon, it's when `data[0]` is 0 (TODO ???) or the entity is `onground` and if we are shooting from a canon, it's when the `actioncooldown` expired. If we are setting `hit` to true, a special case happens when it was false before and `data[0]` is 1 (TODO ???). In that case, the `impactsmoke` particle is played at this object position and if the distance between the player and this object is less than 15.0, the `Thud` sound is played followed by a ShakeScreen using the vector (0.1, 0.35) without reset.

#### `hit` value handling
From there, what happens depends on the value of `hit`.

If it is true:
- The x and z components of the entity's `rigid` velocity are set to `vectordata[0].x` and `vectordata[0].z` respectively
- The entity's `model` angles are incremented by `vectordata[2]` * framestep
- If `data[3]` is present and the NPCControl at the map entity id of its value has its `hit` set to true, then [PlaySound](../../EntityControl/EntityControl%20Methods.md#PlaySound) is called with the `RollingRock` clip looped. If `data[3]` is present, but the `hit` check is violated, the entity's `sound` is stopped. (NOTE: it is possible to test for the `hit` being false by having the negative number in `data[3]`, but a caveat with this is it is only possible to test for the value being false on the id 0, it is NOT possible to test it being true)

If `hit` is false, the clip is stopped as long as it is playing, it exists and the clip is still `RollingRock`. The entity's `sound` is set to no longer loop.

#### Common steps
Regardless of everything, if this object's y position is below `vectordata[1].x` WarpRock is called and its logic depends on if `data[2]` is present and 1 (we are shooting from a canon).

If we are shooting from a canon, [LockRigid(true)](../../EntityControl/EntityControl%20Methods.md#LockRigid) is called on the entity and the `actioncooldown` is reset to `vectordata[1].z`.

If we aren't shooting from a canon:
- This object's transform is set to (0.0, 999.0, 0.0)
- LatePos is called which sets the position of the entity to its `startpos` in `vectordata[1].z` seconds.
- `hit` is set to false
- The entity's `rigid` velocity is zeroed out
- The entity's `onground` is set to false

All of this prepares the next rock to spawn.

## OnTriggerStay
This does nothing if we are in a `pause`, `minipause` or `inevent` and that if a battle is in progress that it's not `inevent`.

If the other gameObject is the player and it's not `digging`, the [event](../../../Enums%20and%20IDs/Events.md) 116 (Getting crushed by a `RollingRock`) with this as the caller.

Otherwise, if the other gameObject tag is `RockLimit`, [BreakRock](../BreakRock.md) is called.

## OnCollisionEnter
If the other gameObject is an `Object` NPCControl of type [BreakableRock](BreakableRock.md), [BreakRock](../BreakRock.md) is called on the other's NPCControl.