# Geizer
A water or honey geizer that can be frozen using Leif's Freeze and it can elevate a [Dropplet](Dropplet.md) ice cube. The oscillation and time to stay frozen are configurable with the option to conditionally activate it when another NPCControl `hit` is true.

## Data Arrays
- `data[0]`: The optional suffix to append to the path when loading the Geizer prefab after converting the number to string, defaults to empty string if not present or if it's <= 0 which is the normal water looking one. NOTE: only a value of 1 is allowed under normal gameplay because it's the only one that gives a valid path (the honey looking geizer), an exception will be thrown if the path is invalid
- `data[1]`: The map entity id whose NPCControl's `hit` should be true for the geizer to be active. This is optional, it can be ommitted or be -1 to have the geizer always be active.
- `data[2]`: If 1 and the map.`lastwater`  exists (the water Hazards), the spout of the main geizer object will continually be set to the map.`lastwater` y position every updates where the geizer isn't frozen
- `data[3]`: If it's not 0, the MeshRenderer of the frozen geizer spout doesn't get enabled when frozen. If it's 1, the geizer position will be set to be entity.`startpos` + the geizer up vector * 10.0 on the first LateUpdate where it is `incamera` and `startlife` is below 20.0 (this means the geizer will start under the floor or inside its wall instead of slowly retracting when the map is done loading). This is optional, it remains enabled if it doesn't exists and nothing special will happen on LateUpdate.
- `data[4]`: If it's not 0, the geizer won't be frozen when it collides with an `IceRadius`. This is optional, it will be frozen if it doesn't exist
- `vectordata[0].x`: 1/6 of the geizer's oscillation's curve frequency in htz
- `vectordata[0].y`: The geizer's oscillation's curve magnitude
- `vectordata[0].z`: The amount of time in frames to keep the geizer frozen when it collides with an ice collider (this is trippled if the Extra Freeze [medal](../../../Enums%20and%20IDs/Medal.md) is equipped)
- `dialogues[2].x`: The local scale of the entity.`model` will be multiplied by the 1/10 of this, but this is optional and the scale will not change if it is 0.1 or below
- `dialogues[2].y`: If not 0, the `electime` of the GlowTrigger when the entity.`originalid` is the  `ElectroPlatform` [AnimID](../../../Enums%20and%20IDs/AnimIDs.md). This is optional and the value defaults is 260.0

## Setup
- entity.`rigid` gets effectively locked by disabling gravity, making it kinematic and freezing all constraints
- isStatic on the gameObject is set to false
- `nointeract` is set to true
- entity.`soundonpause` is set to true
- The gameObject layer to 0 (Default).
- The `scol` is disabled
- `actionfrequency`: is initialised to 2 elemenst :
    - 0: Random.Range(1.0, 10.0) (this is a random phase shift to the geizer oscillation)
    - 1: 0.0 (this is a DeltaTime accumulator for use in the geizer oscillation)
- `actioncooldown`: set to -1100.0 (meaning the first update cycle will be treated as if we had an expired cooldown since more than 1 update cycle)
- A geizer prefab is instantiated from the path `Prefabs/Objects/Geizer` followed by `data[0]` if applicable. The geizer is childed to the entity's `sprite` and has its local position set to Vector3.zero.
- `internaltransform` is initialised to 5 elements:
    - 0: The first child of the geizer's root (The root of the main geizer object)
    - 1: The second child of the geizer's root (the root of the frozen geizer object)
    - 2: The first child of the geizer's first child (the top of the main geizer object)
    - 3: The second child of the geizer's first child (the spout at the bottom of the main geizer's object)
    - 4: The first child of the geizer's second child (the frozen spout at the bottom of the frozen geizer object)
- entity.`model` scale is multiplied by a 1/10 of `dialogues[2].x`
- entity.`alwaysactive` is set to true
- entity.`model` tag is set to `PlatformNoClock`
- If entity.`originalid` is the `ElectroPlatform` [AnimID](../../../Enums%20and%20IDs/AnimIDs.md), a GlowTrigger is added on the first child of the `model`:
    - `getactivecolorfromstart` is set to true
    - `parent` is set to this NPCControl
    - `glowparts` is initialised to a single element corresponding to the MeshRenderer of the first child of the `model`
    - `electime` is initialised to 260.0 unless `dialogues[2].y` exists and isn't 0 where it will take that value instead

## Update
The `internaltransform[3]` (the bottom spout of the main geizer object) gets its local scale set to Vector3.one. If this is the first Update, `initialrender` is initialised to a single element being the MeshRenderer of the spout of the geizer object. This MeshRenderer is then disabled.

From there, the geizer can be active or inactive. It's active if `data[1]` doesn't exist, is -1 or the map entity with the id being `data[1]` exists with a `hit` value of true. It is inactive otherwise. This changes the rest of the update cycle.

### Inactive updates
In the case where the geizer shouldn't be active, GeizerBreak is called if the `actioncooldown` hasn't expired yet (it is also set to 0.0 in this case). This is followed by the position being set to a lerp from the existing one to the entity's `startpos` - the transform's up vector * 10.0 with a factor of 0.025 * the game's frametime. This basically makes the geizer slowly retract itself such that it's not visible because it would be under the floor or inside a wall.

### Active update
If the `startlife` is above 20.0 frames and `hit` is false, `hit` is set to true and the ParticleSystem of `internaltransform[3]` (the spout of the main geizer) is set to play which is some water splashing.

The position is set to lerp from the existing one to the entity.`startpos` with a factor of framestep * 0.025. This makes the geizer slowly show itself if it wasn't active earlier.

From there, there are 3 cases: the `actioncooldown` hasn't expired, it expired last cycle or it has been expired since more than 1 update cycle. This cooldown controls the time left for the geizer to remain frozen before thawing via GeizerBreak. If it expired, it means the geizer isn't frozen.

#### ActionCooldown hasn't expired yet
- If `data[3]` is 0 or doesn't exist, `internalrender[0]` (the MeshRenderer of the spout of the frozen geizer object) gets enabled
- The entity.`sound` is set to not loop anymore
- The `actioncooldown` is decremented by framestep
- The `internaltransform[1]` (the frozen geizer object) local position is set to Vector3.zero unless there's less than 100.0 frames left on the `actioncooldown` in which case, it's set to (Random.Range(-0.05, 0.05), 0.0, 0.0). This applies a somewhat random offset to the frozen geizer every update cycle when there's 100.0 frames left.

#### `actioncooldown` expired since the last Update cycle
If the `actioncooldown` expired since the last cycle (tested by checking it is 0.0 or below, but above -1000.0):

- `internaltransform[0]` is activated (the main geizer object)
- `internaltransform[1]` is deactivated (the frozen geizer object)
- `internaltransform[3]` is activated (the spout at the bottom of main geizer object)
- The `boxcol` is enabled if it is present
- If `startlife` is 15.0 or above, GeizerBreak is called
- The ParticleSystem of `internaltransform[3]` (the spout at the bottom of the water object) is set to play which is some water splashing.
- The entity.`sound` is set to no longer loop
- `actioncooldown` is set to -1100.0 which changes further update cycles to have the cooldown expired since more than 1 cycle.

#### `actioncooldown` expired more than 1 Update cycle ago
If the `actioncooldown` expired more than 1 cycle ago (checked by being below -1000.0):

- `internaltransform[2]` (the top of the main geizer object) is set to rotate by 5.0 degrees in the z axis (this is the vertical one due to the pivot point being rotated)
- `internaltransform[3]` (the spout at the bottom of the water object) is set to rotate by -5.0 degrees in the y axis
- The `boxcol` center is set to (0.0, 3.0 + the root geizer main object y position, 0.0) which elevates it slightly above its position
- The geizer's root position (the parent of `internaltransform[0]`) is set to a lerp from the existing one to the entity. `startpos` + the normalized up vector of the transform * Mathf.Sin(`actionfrequency[1]` * `vectordata[0].x` + `actionfrequency[0]`) * `vectordata[0].y` with a factor of the 1/10 of the game's frametime. This oscillates the geizer's:
    - `vectordata[0].y` is the magnitude of the sine wave, it tells half of the full range between the lowest point the geizer will go and the higest
    - `actionfrequency[1]` is the time in seconds accumulated since SetUp so this scales the Sin through time
    - `actionfrequency[2]` is a phase shift that's been determined randomly on SetUp
    - `vectordata[0].x` is 1/6 of the frequency
- `actionfrequency[1]` is incremented by Time.deltaTime
- If the entity's `sound` wasn't playing, [PlaySound](../../EntityControl/EntityControl%20Methods.md#sounds) is called with the `Waterfall1` clip at 0.075 volume and the entity.`sound` is set to loop
- If `data[2]` is present and 1 and there is a map.lastwater, the y component of the `internaltransform[3]` position (the spout at the bottom of the main geizer object) is set to map.`lastwater` y position. This basically means the spout will be positioned on the map's water Hazards level.
- The `internalrender[0]` (the MeshRenderer of the spout of the frozen geizer object) gets disabled and its position set to the `internaltransform[3]` one (the spout at the bottom of the main geizer object)

## LateUpdate (Not a `dummy`, `incamera` is true and `hit` is false)
- `internaltransform[3]` (the spout of the main geizer object) has its scale set to Vector3.One
- A Raycast is performed from the geizer position + (0.0, 10.0, 0.0) headed down with max 10.0 distance only accepting layers of `Ground` or `NoDigGround`.
- If `attacking` is false (this is the first applicable LateUpdate cycle), it is set to true after a Fader component is added to the gameObject with:
    - forcestayonpause to true
    - childtied to true
    - fadedistance to 0.0
    - pivotoffset to (0.0, the point.y of the collision done earlier or 0.0 if there wasn't a collision - entity.`startpos`, 0.0)
- If `data[3]` is 1 and `startlife` is less than 20.0, the position is set to the entity.`startpos` - the up vector of the geizer * 10.0. This places the geizer such that it will be under the floor / inside its wall very soon after map load
- If there was a collision with the raycast earlier, the position of `internaltransform[3]` (the spout of the main geizer object) is set to the hit point of the raycast and it also gets childed to the entity.`sprite`. This basically ensures the spout is placed where the ground actually is and to ensure it doesn't move with the geizer, it's childed in such a way that it is a sibling of the geizer so it stays there independently of the geizer movements

## OnTriggerEnter
There are 4 branches here:

- The other gameObject tag is `Icecle`, `Icefall` or `IceRadius` and `data[4]` doesn't exist or is 0
- The other gameObject tag is `BeetleHorn` or `BeetleDash` and the `actioncooldown` hasn't expired yet (meaning the geizer is frozen)
- The other gameObject tag is `DroppletCube` and the `actioncooldown`  expired (meaning the geizer isn't frozen)
- The other gameObject tag is `PFollower` or it's the player

### Ice collider logic
- If the `moveobj` is present (meaning a [Dropplet](Dropplet.md) ice cube is on the geizer):
    - LaunchObject is called with it using a random vector between (0.0, -15.0, 0.0) and (0.0, 15.0, 0.0)
    - The parent of the ice cube is set to the current map
    - [ServerGeizer](Dropplet.md#servergeizer) is called onm the `moveobj` Hornable which setups the cube to be on the geizer snapped to it
- If the `actioncooldown` expired (meaning the geizer isn't frozen):
    - The `boxcol` is disabled if present
    - `internaltransform[0]` (the main geizer object) gets disabled
    - `internaltransform[3]` (the spout of the main geizer object) gets disabled
    - `internaltransform[1]` (the frozen geizer object) gets enabled
    - `internaltransform[4]` (the spout of the frozen geizer object) gets enabled except if the current [map](../../../Enums%20and%20IDs/Maps.md) is `UpperSnekGeizerRoom` where it is disabled
    - The entity.`sound` is stopped and placed at the begining of the playback
    - The `Freeze` sound is played at 0.5 volume
- The `actioncooldown` gets set to `vectordata[0].z`, but if the Extra Freeze [medal](../../../Enums%20and%20IDs/Medal.md) is equipped, the value is multiplied by 3 before assigning it

### Kabbu collider logic while the geizer is frozen
- The player entity `hitwall` is set to true
- The `actioncooldown` is set to 0.0
- GeizerBreak is called
- The `boxcol` is disabled if it is present

### [Dropplet](Dropplet.md) cube collider logic while the geizer isn't frozen
- If the `moveobj` is present (there was already a dropplet ice cube on the geizer):
    - LaunchObject is called with it using a random vector between (-5.0, -10.0, 0.0) and (5.0, 10.0, 0.0) 
    - [ServerGeizer](Dropplet.md#servergeizer) is called onm the `moveobj` Hornable which setups the cube to be on the geizer snapped to it
- The RigidBody of the other's parent gets its velocity zeroed out without gravity in kinematic mode
- The parent of the other dropplet is set to `internaltransform[0]` (The main geizer object)
- `moveobj` is assigned to the other dropplet
- The `ingeizer` of the `moveobj`'s Hornable gets set to this `Geizer`

### Player or player follower collider logic
The other gameObject gets rooted to the scene.

## Hazards.OnObjectStay
If the NPCControl passed has this type, it returns true which allows it to stay in collision with the Hazards.

## MainManager.RefreshEntities
This logic only matters if the method was called as a result of MapControl.Start, MapControl.RefreshInsides and BattleControl.ReturnToOverworld.

If `hit` is true and `internaltransform` is not empty, all ParticleSystem under `internaltransform[3]` (the spout at the bottom of the main geizer's object) are played.

## GeizerBreak
This is a private method that only applies to this object type and its job is to shatter a frozen geizer.

- The `IceBreak` sound clip is played at the geizer position + Vector3.Up at 0.75 volume
- `IceShatter` particles are played at (the geizer x position, 0.0 - entity.`startpos` - 0.45, the geizer z position - 0.5) and the scale of those particles is set to (2.0, 3.0, 1.0)
- For all playerdata entities, if any was childed to any transform under this geizer, the entity gets rooted to the scene

## Effects of the `PlatformNoClock` tag
The same than [PathPlatform](PathPlatform.md) except of course, the player entities `noclock` is always false so mostly the childing logic remains. In additiona however, it's what allows [Dropplet](Dropplet.md) ice cube to be childed to the geizer via their Hornable component when the ice cube collides with the geizer.
