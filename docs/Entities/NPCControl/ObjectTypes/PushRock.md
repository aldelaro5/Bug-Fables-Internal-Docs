# PushRock
A rock or a pushable ice cube that can be moved using Kabbu's Horn Slash with configurable movement force. See the simplified explanation section for an intuitive description of how this object functions.

## Data Arrays
- `data[0]`: UNUSED (was used to enable a limited radius via a special collider)
- `data[1]`: If it's 1, the entity.`rigid` constraint are set to none on SetUp, no changes otherwise (This feature is unused under normal gameplay)
- `data[2]`: Determines the mode of operation of the object (This is optional, if it doesn't exist, 0 is assumed):
    - 0: this is a rock that can be moved in any direction by using Kabbu's Horn Slash which launches it in the air before landing
    - 1: An ice cube that can be moved via sliding only in the x and y axises (This is unused under normal gameplay)
    - 2: An ice cube that can be moved via sliding only in the z and y axises (This is unused under normal gameplay)
    - 3: An ice cube that can be moved via sliding only in the x and z axises and that can shatter and respawn if it ever collides with a gameObject having the tag `RockLimit`
- `data[3]`: If it's 1, it is assumed to be a pushable ice cube, but allows all movement axises
- `vectordata[0].x`: UNUSED (was used to specify the scale of the special collider mentioned in `data[0]`)
- `vectordata[0].y`: The y velocity when Horn Slash is used for the launch when this is a rock
- `vectordata[0].z`: The x and z velocity multiplier when Horn Slash is used
- `vectordata[1]`: UNUSED (was used to specify the local position of the special collider mentioned in `data[0]`)

## Additional data
- `boxcol`: Required to be present with valid data as a non trigger collider.

## A simplified explanation of how this object functions
This object is very complex and even includes unused logic under normal gameplay. The underlying system is simpler to explain in its own section which is what this section will attempt to do. If more details are needed, check the other sections.

There are 2 modes to this object which is determined by `data[2]`:

- A rock that can be launched in any directions
- A pushable ice cube that can only be pushed using restricted axises (under normal gameplay, only the y axis is disallowed for movement which is what will be assumed here)

In either cases, `internalcollider[0]` is set to the collider of the entity.`model`

### Rock
When a `BeetleHorn` collides with this while it's not `trapped`, a launch occurs by setting the entity.`rigid` velocity in x/z to match the opposite relative direction of the player with the y being from `vectordata[0].y`. The force of the launch is controlled by a multiplier which is the value of `vectordata[0].z`. As part of the setup, `icevel` is set to the new entity.`rigid`.

During Update, the velocity is managed:

- If the absolute value of the entity.`rigid` y velocity is 0.1 or below (or `onground` became true when absolute y velocity is below 0.15), `icevel` is zeroed out
- The entity.`rigid` x and z velocity are set to the `icevel` ones, but the y is kept

What this means is that while the y velocity is managed by Unity's physics, the x and z ones will zero out whenever the rock has reached low y velocity (which happens at halfway point of the launch) or it has touched the ground. This will eventually cause the rock to remain at rest on the ground.

Worth to note however: this mode has special logic:

- The frictions (static and dynamic) are set to 1.0 or 0.0 depending on if the y velocity is too small in either directions while entity.`onground` is true
- A `Thud` sound occurs when the GroundDetector of the entity sees that the object collided with the ground while it was in the air before, but this can only happen if it's been more than 5.0 frames since the launch

### Pushable ice cube
To keep track of this one, some internal fields are used:

- `internaldata[0]`: The number of frames since the last push
- `internalvector[0]`: The direction the cube should be going (Computed by using CardinalSnap)
- `internalvector[1]`: The last position of the ice cube since the last update cycle

When a `BeetleHorn` collides with this while it's not `trapped`, a push is setup:

- `internalvector[0]` is set to the CardinalSnap of the angle the relative to the player
- `internaldata[0]` is set to 0.0 frames
- `internaldata[1]` to a very high vector (999.0, 999.0, 999.0).

This means the cube will only try to move in x OR z and to the opposite direction of the player's position. The push is registered by setting `hit` to true which changes the Update cycle.

The actual movement happens during Update where it will move by incremented the position by `internalvector[0]` * `vectordata[0].z` * framestep. Everytime this happens, `internaldata[0]` is incremented by framestep.

This keeps going until the ice cube cam't move in the current direction because it's being blocked by a collider or all of the following are true:

- The velocity ramped up for longer than 10.0 frames
- MainManager.FrameDifference returns true (normally returns true, but can skip frames on vsync if the the refresh rate is higher than 60 htz)
- The object moved less than half the x/z multiplier since the last update cycle

When we get there, the entity.`rigid` velocity is zeroed out with `hit` set to false which stops movement and allows another push to happen.

A special case can happen if `data[3]` is 1: it allows the ice cube to move in the y axis too.

## Setup
- The layer is set to 13 (NoDigGround) 
- The `scol` gets disabled
- The `boxcol` gets disabled
- The entity.`alwaysactive` is set to true 
- The `rotater`'s tag is set to `Hornable` which allows PlayerControl to get a green ! emoticon when getting 2.5 or lower distance from the grass for 5 frames
- `internalvector` is initialised to have 2 elements (kept at default values)
- `internaldata` is initialised to have 1 element (kept at default value)
- `internalcollider` is initialised to have 1 element with the value being the entity.`model`'s Collider
- The tags of the gameObject (which gets overriden later to `Object`), the entity.`model` and its child with a Collider are set to `PushRock` which allows this object to actuate a [PressurePlate](PressurePlate.md) and it also allows a DeadLanderOmega to ForceLook if the object collides with a DeadLanderZones whose `onlyrock` is set to true
- The NPCControl's `rotater` is initialised to a new GameObject named `rotater` childed to this object with a local position of Vector3.zero.
- If `data[1]` is 1, the entity.`rigid` have all its constraints removed (this never happens under normal gameplay).
- If `data[2]` is present and above 0, the following happens:
    - The entity's `onground` is set to false
    - The `actioncooldown` is set to 300.0
    - If the `boxcol`.size has a magnitude above 0.1, the first child of the entity's `model` has its scale set to the `boxcol`.size and its local position to (0.0, `boxcol.size.y` / 2, 0.0). The `boxcol` size then gets zeroed out

## Update
If `data[2]` isn't defined or it's 0, PushRockStuff is called (see the section below for details).

Otherwise, the following section occurs.

### `data[2]` is above 0
Some steps occurs depending on the value of `data[2]` (neither happens under normal gameplay):

- 1: The z position is set to the entity.`startpos.z`
- 2: The x position is set to the entity.`startpos.x`

There are 2 subsections, they happen in order if applicable and they aren't mutually exclusive.

#### Ice `hit` logic
This sub section only happens when `hit` is true which can only happen if it was a slidable ice cube. It decides if the cube should be moving or be at rest if it has been moving.

`hit` is set to false and the entity.`rigid` velocity is set to Vector3.zero (which stops its movement and allows to pushed again) if any of the following conditions is true:

- No colliders are detected in a sphere of radius 1.5 with the center being this position + (0.0, 1.0, 0.0) + `internalvector[0]` * 2.0 other than the `ccol`, `boxcol`, `scol`, `pusher` and any colliders in `internalcollider` (Meaning the ice cube cam't move in the current direction because it's being blocked by a collider)
- `data[2]` is between 1 and 2 (never happens under normal gameplay)
- The entity is not `inice` and the map isn't an `icemap`
- All of the following are true
    - `internaldata[0]` is above 10.0 (meaning the velocity ramped up for longer than 10.0 frames)
    - MainManager.FrameDifference returns true (normally returns true, but can skip frames on vsync if the the refresh rate is higher than 60 htz)
    - The distance between this object position and `internalvector[1]` is less than `vectordata[0].z` / 2.0 (meaning the object moved less than half the x/z multiplier since the last update cycle)

Otherwise, the following occurs (meaning the cube should be moving):

- `internalvector[1]` is set to this object's position
- The entity.`rigid` have its rotation frozen, but it is possible to get an additional axis movement constraint frozen if the entity is `onground` or `data[3]` is present and is 1 in addition to:
    - `data[2]` being 1: z movement are frozen (never happens under normal gameplay)
    - `data[2]` being 2, x movement are frozen (never happens under normal gameplay)
    - `data[2]` being 3 and `data[3]` is present and is 1, y movement are frozen
- This object position is incremented by `internalvector[0]` * `vectordata[0].z` * framestep

Finally, `internaldata[0]` is incremented by framestep and the `actioncooldown` is set to 300.0.

#### No ice logic
This sub section only happens when the entity isn't `inice` and the map.`icemap` is false.

The entity.`model` local position is set to Vector3.zero unless there's less than 100.0 frames left to the `actioncooldown` in which case, it's a random vector between (-0.1, 0.0, 0.0) to (0.1, 0.0, 0.0). This slightly shakes the model horizontally.

If the `actioncooldown` hasn't expired, it is decremented by framestep. Otherwise, BreakIceRock is called which does the following (Basically, this respawns the ice cube):

- The `IceShatter` particle is played at this object position + vector3.up with the angles (-90.0, 0.0, 0.0) for 1 second
- The `Audio/Sounds/IceBreak` audio clip is played at this object position with the volume being proportional to the sound distance of the object to the camera scaled by the soundvolume
- The `actioncooldown` is set to 300.0
- The position is set to the entity.`startpos`
- The entity.`onground` is set to false
- The entity.`rigid` gets its rotation frozen

### Common logic
This section happens at the end of the update cycle no matter what.

If the `icevel` magnitude is above 0.1 and the entity.`rigid` velocity is not between -0.1 and 0.1 inclusive, the entity.`rigid` velocity is set to (`icevel`.x, entity.`rigid`.velocity.y, `icevel`.z). Otherwise, the `icevel` is set to Vector3.zero.

### PushRockStuff
- The entity.`activeinevents` is set to true
- The `arrow` is initialised if it was null using HelpArrow.NewArrow with this object as the parent, (0.0, 0.75. 0.0) as the offset, pure green as the color, 2.5 as the distance and 1.5 as the size
- If the `freezecooldown` hasn't expired yet, it is decreased by framestep. Otherwise, the entity.`rigid` x and z velocity are zeroed out.
- If the `actioncooldown` hasn't expired yet, it is decreased by framestep
- If the entity.`rigid` y velocity is between -0.15 and 0.15 while entity.`onground` is true, `icevel` is set to Vector3.zero and both the static and dynamic friction of the `internalcollider[0]` material are set to 1.0. Otherwise, the frictions are set to 0.0 and the entity.`rigid` velocity is overriden to (`icevel`.x, entity.`rigid`.velocity.y, `icevel`.z).

## EntityControl.LateStart
Any object of this type will have [CreateFeet](../../EntityControl/EntityControl%20Methods.md#CreateFeet) called which gives the entity a ground detector in the `feet` field. Additionally, all collisions between `feet`'s Collider and `internalcollider` are ignored.

## EntityControl.UpdateGround
No friction change of the entity.`ccol` can occur with an object of this type because this object's Update can manage it instead.

## OnTriggerEnter
There are 2 branches in this:

- Where the other gameObject tag is `RockLimit` and `data[2]` is 3
- Where the other gameObject tag is `BeetleHorn` and we aren't `trapped`

### `RockLimit`
BreakIceRock is called which does the following:

- The `IceShatter` particle is played at this object position + vector3.up with the angles (-90.0, 0.0, 0.0) for 1 second
- The `Audio/Sounds/IceBreak` audio clip is played at this object position with the volume being proportional to the sound distance of the object to the camera scaled by the soundvolume
- The `actioncooldown` is set to 300.0
- The position is set to the entity's `startpos`
- The entity's `onground` is set to false
- The entity's `rigid` gets its rotation frozen

### `BeetleHorn`
The `Damage0` sound is played on the entity at 0.6 volume and 0.5 pitch if the entity `sound` wasn't played anything or it was playing, but the clip wasn't `Damage0` or the playback of the existing sound is more than halfway done.

Then, HitPart particles are played at this position + (0.0, 0.5, 0.0) and the `rotater` is set to LookAt the player.

After, there are 2 kinds of logic there and it depends on `data[2]` existing and either the entity.`onground` is true or `data[3]` exists and is 1. Either starts a movement on the rock or ice cube

If the above is violated (This is a rock):

- The entity.`rigid` x and z velocity are set to be 0.0 - `rotater.forward.x` and 0.0 - `rotater.forward.z` respectively, each scaled by `vectordata[0].z`. The y velocity is set to `vectordata[0].y`
- `icevel` is set to the entity.`rigid` velocity
- The entity `onground` is set to false
- The `actioncooldown` is set to 5.0
- The `freezecooldown` is set to 150.0
- The entity `feet.overridecd` is set to 5.0

If it was fufilled (This is an ice cube):

- `internaldata[0]` is set to 0.0
- `internalvector[1]` is set to (999.0, 999.0, 999.0)
- What happens next depends on `data[2]`:
    - If it's 0, the same logic occurs than if `data[2]` didn't existed in the first place
    - If it's 1 or 2, `hit` is set to true (Never happens under normal gameplay)
    - If it's 3, `internalvector[0]` is set to the CardinalSnap of the `rotater` and `hit` is set to true 

## GroundDetector.OnTriggerStay
If the object reached the ground while it was in the air before and `actioncooldown` expired, DeathSmoke particles are played at the object position + the forward vector of the camera * -0.35 followed by a `Thud` sound played on the entity at 0.4 volume and 1.5 pitch.

## PauseMenu.NearSomething
If the method finds out that a map entity of this object type exists less than 4.0 distance from the player, the return will be an array of 3 booleans with the first one being true. This will prevent usage of the Bed Bug [item](../../../Enums%20and%20IDs/Items.md).

## MainManager.RefreshEntities
Strangely, the logic there doesn't end up doing anything useful because it ensures the entity.`rigid` has its gravity enabled without kinematic, but there is no situation in which this won't be true because it's the default of the RigidBody and it's also never changed.