# Dropplet
A water dropple that can be frozen by Leif's Freeze.

## Data Arrays
- `data[0]`: ???
- `data[1]`: ???
- `data[2]`: If 1, ??? (involved with shadows)
- `data[3]`: A cooldown ???
- `vectordata[0]`: ??? (incremented by the entity's `startpos` on SetUp)
- `vectordata[1].x`: The x component of the pushamount of the Hornable of the ice cube
- `vectordata[1].y`: The y component of the pushamount of the Hornable of the ice cube
- `vectordata[1].z`: The volume multiplier of the `WaterSplash` sound when a dropplet collides with a `Ground` or `NoDigGround`

## Setup
First, the `actionfrequency` is initialised to 3 elements left at 0.0 each.

Then, the entity's `hasshadow`, `overrideshadow`, `alwaysactive` and `activeonpause` are set to true. The `shadow` is ensured to be created if it wasn't.

From there, `internaltransform` is initialised to one element which is an instance of the `Prefabs/Objects/icecube` prefab which is then childed to the map. Some adjustements are performed on it:
- The layer is set to 13 (NoDigGround)
- The local scale is set to Vector3.one * 1.5
- A RigidBody is added on it with kinematic and rotation frozen with a mass of 100000.0
- The BoxCollider present at the root of it gets enabled without trigger at size Vector3.one
- A DialogueAnim is added with a startsize and targetpos of Vector3.1.5, a targetpos of Vector3.zero and a speed of 0.01
- A Hornable is added with a push of (`vectordata[1].x`, `vectordata[1].y`) created from this NPCControl with breakondash and without pusher.
- The first child's tag is set to DroppletCube (this is the actual cube inside the outer BoxCollider)
- A trigger BoxCollider is added to the DroppletCube with a size of Vector3.one * 0.01. All collisions with it and the player's `ccol` or the outer BoxCollider are ignored

Then, `vectordata[0]` is incremented by the entity's `startpos`.

`internalparticle` is initialised to a single element which is the SpriteBounce of a instance of `Prefabs/Objects/WaterBubble` which is then childed to the entity's `sprite` and has a local position of Vector3.up / 2.

Finally, the entity's `rigid` gets its gravity enabled with rotation frozen and its `onground` set to false.

## Update
First, if the entity's `rigid` isn't in kinematic mode, its y velocity will be set to -10.0 and [RefreshShadow](../../EntityControl/Update%20process/RefreshShadow.md) is called on the entity. This forces the dropplet to drop down at a constant speed.

If the `actioncooldown` hasn't expired yet, it is decremented by the game's frametime. Otherwise [LockRigid(false, false)](../../EntityControl/EntityControl%20Methods.md#LockRigid) is called on the entity. This allows to unlock the entity's `rigid` once a dropplet is ready to be dropped.

If the entity's `shadow` is present, it will get enabled if `data[2]` is 0 or `hit` is false and it will get disabled otherwise.

If the `actionfrequency[2]` cooldown hasn't expired yet and we aren't on `pause`, it will be decremented by framestep. Otherwise, this is the main part of the update cycle where this cooldown expired. TODO: that thing is not reset here, why ???

TODO: the logic in here has a lot of stuff that probably gets supplemented in trigger handling and it's not possible to figure out everything based on just update ???

## LateUpdate (Non `dummy`, the entity is `incamera` and not `iskill`)
Normally under these circumstances, If the y position is less than the map.`ylimit`, the the position is set to the entity `startpos` and DeathSmoke particles are played at the entity `sprite` position if it isn't `dead` and the [animid](../../Enums%20and%20IDs/AnimIDs.md) isn't negative (it is defined).

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
  - If `data[3]` exists and is above 0, `actioncooldown` is set to it
  - The entity `rigid` is put in kinematic mode
  - The position is set to offscreen at (0.0, -2000.0, 0.0) in 0.1 seconds
- `actionfrequency[2]` is set to `data[0]`

## OnTriggerEnter if the other gameObject layer is `Ground` or `NoDigGround`
This does nothing if the other gameObject tag is `DroppletPass`.

- `actionfrequency[1]` is set to 600.0
- If the entity `rigid` is not in kinematic mode, the following depends on `internalparticle[0]`:
  - If it is null, it is initialised to a `WaterSplash` ParticleSystem being played at this object position with angles of (-90.0, 0.0, 0.0) and it is then parented to the map.
  - If it wasn't null, its position is set to this object position and Play is called on it
- If we aren't in a `pause` and the `startlife` is over 60.0 frames, the `WaterSplash` sound is played at this object position with a volume of the entity sound distance adjusted for the volume settings and multipled by `vectordata[1].z` if it wasn't 0.0.
- The entity `rigid` is put in kinematic mode
- The position is set to be offscreen at (0.0, -2000.0, 0.0)
- `actionfrequency[0]` is set to -1110.0 TODO ???
- `actionfrequency[2]` is set to `data[0]` TODO ???
