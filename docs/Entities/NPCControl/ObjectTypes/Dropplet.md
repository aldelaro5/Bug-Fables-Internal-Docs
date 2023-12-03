# Dropplet
A water dropplet that can be frozen by Leif's Freeze which then turns it into an ice cube that can be moved by launching it using Kabbu's Horn Slash. The push velocity vector when launched is configurable and the object also supports many optional features, see the data array definition for details.

## Data Arrays
- `data[0]`: The time interval in frames between main Updates where the ice cube and dropplet are managed. This allows to throttle those updates
- `data[1]`: If above 0, the max distance in units the cube can away of entity.`startpos` + `vectordata[0]`. If the ice cube goes further, it will be shattered via ShatterDroppletIce. This isn't enforced if the value is 0 or below
- `data[2]`: There are 3 possible meanings:
  - If 0, entity.`shadow` is always kept enabled and when the dropplet is frozen, its position is set to be offscreen immediately
  - If 1, entity.`shadow` is only enabled when `hit` is false (no ice cube exists) and when the dropplet is frozen, the entity.`rigid` is put in kinematic mode and the dropplet position is set to be offscreen in 0.1 seconds. This also allows `data[3]` to take effect if applicable
  - Any other value: If 1, entity.`shadow` is only enabled when `hit` is false (no ice cube exists) and when the dropplet is frozen, its position is set to be offscreen immediately
- `data[3]`: If `data[2]` is 1 and this value is above 0, the time period in frames that the ice cube is allowed to exist before shattering via ShatterDroppletIce on its own. This is optional, this feature is disabled if it doesn't exist or is 0 or below. NOTE: specifying a value above 0 while `data[2]` isn't 1 is considered invalid and will cause the ice cube to shatter on the first throttled update
- `data[4]`: If 1, the ice cube can only be launched in the 8 cardinal directions relative to the camera instead of having free movement in x and z
- `vectordata[0]`: If `data[1]` is above 0, this is an offset position where its value + `startpos` is the center of the range where the ice cube is allowed to stay with a max distance of `data[1]`. If the ice cube goes out of this range, it is shattered via ShatterDroppletIce
- `vectordata[1].x`: The x and z velocity of the ice cube when launched
- `vectordata[1].y`: The y velocity of the ice cube when launched
- `vectordata[1].z`: If not 0.0, the volume multiplier of the `WaterSplash` sound when a dropplet collides with a GameObject with layer `Ground` or `NoDigGround`

## Setup
- `actionfrequency` is initialised to 3 elements left at 0.0 each
- entity.`hasshadow` is set to true
- entity.`overrideshadow` is set to true
- entity.`alwaysactive` is set to true
- entity.`activeonpause` is set to true
- entity.`shadow` is ensured to be created if it wasn't

From there, `internaltransform` is initialised to one element which is an instance of the `Prefabs/Objects/icecube` prefab which is then childed to the map. Some adjustements are performed on it:
- The layer is set to 13 (NoDigGround)
- The local scale is set to Vector3.one * 1.5
- A RigidBody is added on it with kinematic and rotation frozen with a mass of 100000.0
- The BoxCollider present at the root of it gets enabled without trigger at size Vector3.one
- A DialogueAnim is added with a startsize and targetpos of Vector3 * 1.5, a targetpos of Vector3.zero and a speed of 0.01
- A Hornable is added with a push of (`vectordata[1].x`, `vectordata[1].y`) created from this NPCControl with breakondash and without pusher.
- The first child's tag is set to `DroppletCube` (this is the actual cube inside the outer BoxCollider)
- A trigger BoxCollider is added to the `DroppletCube` with a size of Vector3.one * 0.01. All collisions with it and the player.entity.`ccol` or the outer BoxCollider are ignored

After:
- `vectordata[0]` is incremented by the entity.`startpos`
- `internalparticle` is initialised to a single element which is the SpriteBounce of a instance of `Prefabs/Objects/WaterBubble` which is then childed to the entity.`sprite` and has a local position of Vector3.up / 2
- entity.`rigid` gets its gravity enabled with rotation frozen and entity.`onground` set to false

## Update
If the entity.`rigid` isn't in kinematic mode, its y velocity will be set to -10.0 and [RefreshShadow](../../EntityControl/Update%20process/RefreshShadow.md) is called on the entity. This forces the dropplet to drop down at a constant speed.

If the `actioncooldown` hasn't expired yet, it is decremented by the game's frametime. Otherwise [LockRigid(false, false)](../../EntityControl/EntityControl%20Methods.md#LockRigid) is called on the entity. This allows to unlock the entity.`rigid` once a dropplet is ready to be dropped.

If the entity.`shadow` is present, it will get enabled if `data[2]` is 0 or `hit` is false and it will get disabled otherwise.

What follows is logic that depends on the `actionfrequency` 3 different cooldowns.

### `actionfrequency` cooldowns
Here is what the different array elements mean:
- `actionfrequency[0]`: UNUSED (It always stays to -1100 and never changes)
- `actionfrequency[1]`: The max cooldown before the dropplet is moved to entity.`startpos` which is hardcoded to 600.0
- `actionfrequency[2]`: The cooldown before doing any of this logic which means it serve as a throttle. Defaults to `data[2]`

Nothing in this section happens if `actionfrequency` hasn't expired yet besides the value being decremented by framestep. This effectively throttles main updates to only happen every `data[0]` frames.

If we aren't being throttled, this is where `actionfrequency[1]` comes into play. If it hasn't expired yet and the entity y position is -25.0 or above, then `actionfrequency[1]` is decremented by framestep. Otherwise, it means the cooldown expired, the dropplet dropped to \< -25 or it was placed offscreen by OnTriggerEnter as a result of touching the ground or being frozen. In that case, the dropplet is reset back to its starting position by doing the following:
- The position is set to entity.`startpos`
- `actionfrequency[1]` is reset to 600.0 which allows the cycle to restart
- entity.`rigid` velocity gets zeroed out without kinematic mode

From there the following always happen:
- [LockRgid(false)](../../EntityControl/EntityControl%20Methods.md#lockrigid) is called on the entity
- entity.`rigid` gets its rotation frozen

If `hit` is false or we are in a `pause` or `minipause`, this section is done as the rest only concerns the ice cube when unpaused.

If `data[1]` is above 0 and `internaltransform[0]` (the actual ice cube) is more than `data[1]` away from entity.`startpos` + `vectordata[0]`, it is shattered via ShatterDroppletIce.

We only proceed if `data[3]` exists and is above 0.

This is where the new `actioncooldown` value is managed. It was already decremented earlier so the only logic here is when it goes below 100.0 every 3 frames and when it expires.

If it's below 100.0 every 3 frames:
- The `IceMelt` sound effect is played at `internaltransform[0]` (the actual ice cube) with the volume being that transform's sound distance * 2.0 adjusted by the game's volume settings.
- The DialogueAnim of the ice cube targetscale is set to Vector3.zero to make it shrink to nothing at a speed of 0.01

If it's expired, the ice cube is shattered via ShatterDroppletIce and ServerGeizer is called on the ice cube's Hornable.

## LateUpdate (Non `dummy`, the entity is `incamera` and not `iskill`)
Normally under these circumstances, If the y position is less than the map.`ylimit`, the the position is set to the entity.`startpos` and DeathSmoke particles are played at the entity.`sprite` position if it isn't `dead` and the [animid](../../Enums%20and%20IDs/AnimIDs.md) isn't negative (it is defined).

However, `Dropplet` are exempt from the logic above so they aren't bound by the map.`ylimit`.

## OnTriggerEnter main logic
This does nothing if the other gameObject tag isn't `Icecle` or `IceRadius`.

If `hit` is true, the `IceShatter` particle is played at the `internaltransform[0]` position and the `IceBreak` sound is played at the same position with the volume being the sound distance of the position adjusted for the game's volume * 1.5. Finally, it also causes ServerGeizer to be called on the `internaltransform[0]`'s Hornable.

The following occurs:
- `hit` is set to true
- `actionfrequency[0]` is set to -1100.0
- `actionfrequency[1]` is set to 600.0
- `internaltransform[0]` gets some adjustements:
  - Its position gets set to the other position + (0.0, 0.25, 0.0)
  - It gets childed to the map
  - Its local scale gets set to Vector3.zero
  - Its Hornable `ingeizer` gets set to null
  - Its DialogueAnim is enabled with a `shrinkspeed` of 0.1 and a `targetscale` of (1.5, 1.5, 1.5)
  - Its RigidBody gets its velocity zeroed out and put out of kinematic mode
- If the `collisionammount` is less than 2, the `Freeze` sound is played at 0.6 volume
- `collisionammount` is incremented
- If `data[2]` is 1 (otherwise, the position is set to offscreen at (0.0, -2000.0, 0.0)):
  - If `data[3]` exists and is above 0, `actioncooldown` is set to `data[3]`
  - The entity.`rigid` is put in kinematic mode
  - The position is set to offscreen at (0.0, -2000.0, 0.0) in 0.1 seconds
- `actionfrequency[2]` is set to `data[0]`

## OnTriggerEnter if the other gameObject layer is `Ground` or `NoDigGround`
This does nothing if the other gameObject tag is `DroppletPass` (This would have happened with a [BreakableRock](BreakableRock.md), but the tag gets overriden to `Object` later so in practice, this can't happen under normal gameplay).

- `actionfrequency[1]` is set to 600.0
- If the entity.`rigid` is not in kinematic mode, the following depends on `internalparticle[0]`:
  - If it is null, it is initialised to a `WaterSplash` ParticleSystem being played at this object position with angles of (-90.0, 0.0, 0.0) and it is then parented to the map.
  - If it wasn't null, its position is set to this object position and Play is called on it
- If we aren't in a `pause` and the `startlife` is over 60.0 frames, the `WaterSplash` sound is played at this object position with a volume of the entity sound distance adjusted for the volume settings and multipled by `vectordata[1].z` if it wasn't 0.0.
- The entity.`rigid` is put in kinematic mode
- The position is set to be offscreen at (0.0, -2000.0, 0.0)
- `actionfrequency[0]` is set to -1110.0
- `actionfrequency[2]` is set to `data[0]`

## MainManager.RefreshEntities
This logic only matters if the method was called as a result of MapControl.Start, MapControl.RefreshInsides and BattleControl.ReturnToOverworld.

If `data[2]` is 0 or `hit` is false:
- entity.`rigid` gets its velocity zeroed without kinematic and with gravity
- entity.`onground` is set to false
- If entity.`originalmap` is the current one, the dropplet is childed to the map

## ShatterDroppletIce
This is a public method that is practically called in a wide variety of places, but it only contains logic pertinent to this object type. It shatters the ice cube by doing the following:
- `hit` is set to false which makes the game no longer recognise the ice cube exists
- `IceShatter` particle is played at the ice cube's position + (0.0, 1.0, 0.0) with angles (-90.0, 0.0, 0.0) for 1.0 second
- The `IceBreak` sound effect is played at the ice cube's position with the volume being the ice cube sound distance adjusted by the game volume settings
- The position of the ice cube is set to be offscreen (0.0, -1000.0, 0.0) the next frame
- The RigidBody of the ice cube is put into kinematic mode
- The ice cube is childed to the map
- The DialogueAnim of the ice cube is enabled

## Hornable
This component is only used for this object type on `internaltransform[0]` (the actual ice cube) making all its logic exclusive to a dropplet. Its job is to manage the actual movement of the ice cube when using Kabbu's Horn Slash or when it is on a [Geizer](Geizer.md).

For the sake of brevety, the component won't be fully documented, but a summary of the methods is provided here.

To properly add the Hornable, SetUp must be called and in practice, it's always called with the same parameters on this NPCContol's SetUp:
- pushammount is (`vectordata[1].x`, `vectordata[1].y`)
- pusherenabled is false, but then overriden to true on the component's Start (prevents it to be affected by a `pusher` collider on an [NPC](../NPCType.md#npc))
- dashbreak is true (this is unused so it does nothing)
- parent is this NPCControl

On its Start, the main thing that happens is the tag is set to `Hornable` (allows PlayerControl to get a green ! emoticon when getting 2.5 or lower distance from the grass for 5 frames), a HelpArrow is setup with a cyan color and a rotater object is created childed to the ice cube (this is used later when `data[4]` gets involved).

On its LateUpdate, the main things that happens are:
- Some logic where the ice cube is visually displaced when the ice cube is in a [Geizer](Geizer.md)
- The frictions are managed here (basically full friction when at rest, no frictions otherwise)
- Enforcing that the cube cannot be more than 150.0 frames in the air or its x and z velocity will be zeroed out and forced to be on the ground

On its OnTriggerEnter, there are multiple possible interactions depending on the other collider tag:
- `BeetleHorn`: This means the cube will be launched. The launch is done by using pushammount.x for the x and z axises and pushammount.y for the y axis as the new velocity adjusted by the actual direction the player is at so it launches away from it. However, this can be skewed if `data[4]` exists and is 1 where the actual direction is snapped to the closest 8 cardinal directions relative to the camera (this is done using the rotater object as it's set to look at the player)
- `IceBreak`: the player.entity.`hitwall` is set to true and ShatterDroppletIce is called on the NPCControl
- `Platform` or `PlatformNoClock`: The ice cube can get childed to the platform (or [Geizer](Geizer.md)) so it moves with it. This is also done in OnCollisionEnter. This is undone in OnTriggerExit when applicable

On its OnTriggerStay, any `Pusher` collider is enforced here and the ground state is reset when the other collider is layer 8 or 13 (`Ground` or `NoDigGround`). This is also done in OnCollisionEnter and OnCollisionStay

### ServerGeizer
This public method of the component is important because it setup some changes when the ice cube gets on a [Geizer](Geizer.md). It does nothing however if `ingeizer` is null.

- The collisions between the root collider or the actual ice cube collider and the ingeizer.`boxcol` are temporarilly ignored for 5.0 seconds
- The ice cube is childed to the map
- The root RigidBody velocity is zeroed out with gravity and without kinematic mode
- ingeizer.`moveobj` is set to null
- ingeizer is set to null