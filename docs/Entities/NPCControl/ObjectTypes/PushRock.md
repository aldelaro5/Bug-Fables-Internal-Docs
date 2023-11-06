TODO: in general how does this work ???

# PushRock
A rock that can be moved using Kabbu's Horn Slash ???

## Data Arrays
- `data[0]`: ???
- `data[1]`: If it's 1, the entity's `rigid` constraint are set to none, no changes otherwise
- `data[2]`: Between 0 and 3 ??? (this is optional)
- `data[3]`: If it's 1 ???
- `vectordata[0].x`: ???
- `vectordata[0].y`: The y velocity when Horn Slash is used on the rock when ???
- `vectordata[0].z`: The x and z velocity multiplier when Horn Slash is used on the rock when ???

## Setup
First, the tag is set to `PushRock`, the layer to 13 (NoDigGround) and the `scol` gets disabled. The entity's `alwaysactive` is set to true and its `rotater` tag to `Hornable`.

From there, `internalvector` is initialised to have 2 elements and `internaldata` to have 1, but they are kept at default values.

The entity's `model` and all its child gets the `PushRock` tag. `internalcollider` is also initialised to have a single element being the Collider of the entity's `model`.

The NPCControl's `rotater` is initialised to a new GameObject named `rotater` childed to this object with a local position of Vector3.zero.

If `data[2]` is present and above 0, the following happens:
- The entity's `onground` is set to false
- The `actioncooldown` is set to 300.0
- If the `boxcol`'s size has a magnitude above 0.1, the first child of the entity's `model` has its scale set to the `boxcol`'s size and its local position to (0.0, `boxcol.size.y` / 2, 0.0). The `boxcol` size then gets zeroed out

Finally, the `boxcol` gets disabled.

## Update
If `data[2]` isn't defined, PushRockStuff is called (see the section below for details).

Otherwise, the following section occurs.

### `data[2]` defined logic
Some steps occurs depending on the value of `data[2]`:
- 0: PushRockStuff is called (check the section below about it)
- 1: The z position is set to the entity's `startpos.z`
- 2: The x position is set to the entity's `startpos.x`

From there, this section only continues if `data[2]` is above 0, otherwise, this cycle continues to the common logic section. 

There are 2 subsections, they happen in order if applicable and they aren't mutually exclusive.

#### `hit` logic
This sub section only happens when `hit` is true.

`hit` is set to false and the entity's `rigid` velocity is set to Vector3.zero if any of the following conditions are true:
- No colliders are detected in a sphere of radius 1.5 with the center being this position + (0.0, 1.0, 0.0) + `internalvector[0]` * 2.0 other than the `ccol`, `boxcol`, `scol`, `pusher` and any colliders in `internalcollider`
- `data[2]` is less than 3
- The entity is not `inice` and the map isn't an `icemap`
- All of the following are true
  - `internaldata[0]` is above 10.0
  - MainManager.FrameDifference ??? (TODO: this looks buggy, recheck)
  - The distance between this object position and `internalvector[1` is less than `vectordata[0].z` / 2.0
  - Any of the following is true:

Otherwise, the following occurs:
- `internalvector[1]` is set to this object's position
- The entity's `rigid` have its rotation frozen, but it is possible to get an additional axis movement constraint frozen if the entity is `onground` or `data[3]` is present and 1 in addition to:
  - `data[2]` not being 1: z movement are frozen
  - `data[2]` not being 1, x movement are frozen
  - `data[3]` not being present or isn't 1 and `data[2]` is 2: y movement are frozen
- This object position is incremented by `internalvector[0]` * `vectordata[0].z` * framestep

Finally, `internaldata[0]` is incremented by framestep and the `actioncooldown` is set to 300.0.

#### No ice logic
This sub section only happens when the entity isn't `inice` and the map's `icemap` is false.

If the entity's `model` is present, its local position is set to Vector3.zero unless there's less than 100.0 frames left to the `actioncooldown` in which case, it's a random vector between (-0.1, 0.0, 0.0) to (0.1, 0.0, 0.0). This slightly shakes the model horizontally.

If the `actioncooldown` expired, BreakIceRock is called, check the section below for more details (otherwise, the `actioncooldown` is decremented by framestep)

### Common logic
This section happens at the end of the update cycle no matter what.

If the `icevel` magnitude is above 0.1 and the entity's `rigid` velocity is not between -0.1 and 0.1 inclusive, the entity's `rigid` velocity is set to (`icevel.x`, the entity's `rigid` y velocity, `icevel.z`). Otherwise, the `icevel` is set to Vector3.zero.

### PushRockStuff
TODO

## OnTriggerEnter
There are 2 branches in this:
- Where the other gameObject tag is `RockLimit` and `data[2]` is 3
- Where the other gameObject tag is `BeetleHorn` and we aren't `trapped`

### `RockLimit`
BreakIceRock is called, check the section below for details

### `BeetleHorn`
First, the entity `rigid` is put out of kinematic mode with gravity. Then, the `Damage0` sound is played on the entity at 0.6 volume and 0.5 pitch if the entity `sound` wasn't played anything or it was playing, but the clip wasn't `Damage0` or the playback of the existing sound is more than halfway done. Then, HitPart particles are played at this position + (0.0, 0.5, 0.0) and the `rotater` is set to LookAt the player.

After, there are 2 kinds of logic there and it depends on `data[2]` existing and either the entity `onground` is true or `data[3]` exists and is 1.

If the above is violated:
- The entity `rigid` x and z velocity are set to be 0.0 - `rotater.forward.x` and 0.0 - `rotater.forward.z` respectively, each scaled by `vectordata[0].z`. The y velocity is set to `vectordata[0].y`
- `icevel` is set to the `rigid` velocity
- The entity `onground` is set to false
- The `actioncooldown` is set to 5.0
- The `freezecooldown` is set to 150.0
- The entity `feet.overridecd` is set to 5.0

If it was fufilled:
- `internaldata[0]` is set to 0.0
- `internalvector[1]` is set to (999.0, 999.0, 999.0)
- What happens next depends on `data[2]`:
  - If it's 0, the following occurs:
    - The entity `rigid` x and z velocity are set to be 0.0 - `rotater.forward.x` and 0.0 - `rotater.forward.z` respectively, each scaled by `vectordata[0].z`. The y velocity is set to `vectordata[0].y`
    - `icevel` is set to the `rigid` velocity
    - The entity `onground` is set to false
    - The `actioncooldown` is set to 5.0
    - The `freezecooldown` is set to 150.0
    - The entity `feet.overridecd` is set to 5.0
  - If it's 1 or 2, `hit` is set to true
  - If it's 3, `internalvector[0]` is set to the CardinalSnap of the `rotater` and `hit` is set to true 

## BreakIceRock
- The `IceShatter` particle is played at this object position + vector3.up with the angles (-90.0, 0.0, 0.0) for 1 second
- The `Audio/Sounds/IceBreak` audio clip is played at this object position with the volume being proportional to the sound distance of the object to the camera scaled by the soundvolume
- The `actioncooldown` is set to 300.0
- The position is set to the entity's `startpos`
- The entity's `onground` is set to false
- The entity's `rigid` gets its rotation frozen