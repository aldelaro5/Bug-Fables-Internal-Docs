# Geizer
A water or honey geizer that can be frozen using Leif's Freeze.

## Data Arrays
- `data[0]`: The optional suffix to append to the path when loading the Geizer prefab after converting the number to string, defaults to empty string if not present or if it's <= 0 which is the normal water looking one. NOTE: only a value of 1 is allowed under normal gameplay because it's the only one that gives a valid path (the honey looking geizer), an exception will be thrown if the path is invalid
- `data[1]`: The map entity id whose NPCControl's `hit` should be true for the geizer to be active. This is optional, it can be ommitted or be -1 to have the geizer always be active.
- `data[2]`: If 1, ??? (TODO: involves map.latwater). This is optional
- `data[3]`: If it's 0, the MeshRenderer of the geizer's top part gets enabled when frozen (TODO this is very unclear ???) This is optional.
- `data[4]`: If it's 0, ???
- `vectordata[0].x`: ??? (has to do with the geizer oscillation)
- `vectordata[0].y`: ??? (has to do with the geizer oscillation)
- `vectordata[0].z`: The amount of time in frames to keep the geizer frozen when using Freeze or Icicle

## Setup
First, the entity's `rigid` gets effectively locked by disabling gravity, making it kinematic and freezing all constraints. The isStatic on the gameObject is set to false, the `nointeract` to true and the entity's `soundonpause` to true.

Them, the enity's `alwaysactive` is set to true and the gameObject layer to 0 (Default).

After, some NPCControl fields adjustements are peformed:
- `actionfrequency`: overriden to {Random.Range(1, 10), 0.0}
- `actioncooldown`: set to -1100.0 (meaning the first update cycle will be treated as if we had an expired cooldown since more than 1 update cycle)

From there, a geizer prefab is instantiated from the path `Prefabs/Objects/Geizer` followed by `data[0]` if applicable. The geizer is childed to the entity's `sprite` and has its local position set to Vector3.zero.

Finally, the `internaltransform` array is initialised to 5 elements:
- 0: The first child of the geizer's root (the water object which is the normal looking geizer)
- 1: The second child of the geizer's root (the frozen geizer object)
- 2: The first child of the geizer's first child (the top of the main geizer object)
- 3: The second child of the geizer's first child (the spout at the bottom of the main geizer's object)
- 4: The first child of the geizer's second child (the frozen spout at the bottom of the frozen geizer object)

## Update
First, the `internaltransform[3]` (the bottom part of the geizer object) gets its local scale set to Vector3.one. If this is the first Update, `initialrender` is initialised to a single element being the MeshRender of the top part of the geizer object. This Meshrenderer is then disabled.

The main update cycle occurs if the geizer is active. It is active if the `data[1]` check doesn't apply or it does, but it has been fufilled. In the case where the geizer shouldn't be active, [GeizerBreak](../GeizerBreak.md) is called if the `actioncooldown` hasn't expired yet (it is also set to 0.0 in this case). This is followed by the position being set to a lerp from the existing one to the entity's `startpos` - the transform's up vector * 10.0 with a factor of 0.025 * the game's frametime. TODO: why are we moving the positon? is the position the top of the geizer ???

If the geizer is active, the main logic of this update cycle follows.

### Active update
First, if the `startlife` is above 20.0 frames and `hit` is false, `hit` is set to true and the ParticleSystem of `internaltransform[3]` (the bottom part of the geizer) is set to play which is some water splashing.

Then, the position is set to lerp from the existing one to the entity's `startpos` with a factor of framestep * 0.025. TODO: again, why are we doing this ???

From there, there are 3 cases: the `actioncooldown` hasn't expired, it expired last cycle or it has been expired since more than 1 update cycle.

If it hasn't expired yet:
- `internalrender[0]` (the MeshRenderer of the top part of the geizer object) gets enabled if `data[3]` is 0 and present
- The entity's `sound` is set to not loop anymore
- The `actioncooldown` is decremented by framestep
- The `internaltransform[1]` (the frozen geizer object) local position is set to Vector3.zero unless there's less than 100.0 frames left on the `actioncooldown` in which case, it's set to (Random.Range(-0.05, 0.05), 0.0, 0.0). This applies a somewhat random offset to the frozen geizer every update cycle when there's 100.0 frames left.

If the `actioncooldown` expired last cycle (tested by checking it is below 0.0, but above -1000.0):
- `internaltransform[0]` is activated (the water object)
- `internaltransform[1]` is deactivated (the frozen geizer object)
- `internaltransform[3]` is activated (the spout at the bottom of the water object)
- The `boxcol` is enabled if it is present
- If `startlife` is 15.0 or above, [GeizerBreak](../GeizerBreak.md) is called
- The ParticleSystem of `internaltransform[3]` (the spout at the bottom of the water object) is set to play which is some water splashing.
- The entity's `sound` is set to no longer loop
- `actioncooldown` is set to -1100.0 which changes further update cycles to have the cooldown expired since more than 1 cycle.

If the `actioncooldown` expired more than 1 cycle ago:
- `internaltransform[2]` (the top of the water object) is set to rotate by 5.0 degrees in the z axis
- `internaltransform[3]` (the spout at the bottom of the water object) is set to rotate by -5.0 degrees in the y axis
- The `boxcol` center is set to (0.0, 3.0 + the root geizer's object y position, 0.0) which elevates it slightly above its position
- The geizer's root position is set to a lerp from the existing one to the entity's `startpos` + the normalized up vector of the transform * `Mathf.Sin(actionfrequency[1] * vectordata[0].x + actionfrequency[0]` * `vectordata[0].y` with a factor of the 1/10 of the game's frametime. TODO: what is this math ???
- `actionfrequency[1]` is incremented by Time.deltaTime
- If the entity's `sound` wasn't playing, [PlaySound](../../EntityControl/EntityControl%20Methods.md#PlaySound) is called with the `Waterfall1` clip at 0.075 volume
- If `data[2]` is present and 1 and there is a map.lastwater, the y component of the `internaltransform[3]` position (the spout at the bottom of the water object) is set to map.lastwater's y position TODO: huh ???
- The `internalrender[0]` (the MeshRenderer of the top part of the geizer object) gets disabled and its position set to the `internaltransform[3]` one (the spout at the bottom of the water object)

## OnTriggerEnter
There are 4 branches here:
- The other gameObject tag is `Icecle`, `Icefall` or `IceRadius` and `data[4]` doesn't exist or is 0
- The other gameObject tag is `BeetleHorn` or `BeetleDash` and the `actioncooldown` hasn't expired yet
- The other gameObject tag is `DroppletCube` and the `actioncooldown` hasn't expired yet
- The other gameObject tag is `PFollower` or it's the player

### Ice collider logic
- If the `moveobj` is present:
  - LaunchObject is called with it using a random vector between (0.0, -15.0, 0.0) and (0.0, 15.0, 0.0)
  - The parent of the `moveobj` is set to the current map
  - ServerGeizer is called onm the `moveobj` Hornable which setups the cube to be on the geizer snapped to it
- If the `actioncooldown` expired:
  - The `boxcol` is disabled if present
  - `internaltransform[0]` and `internaltransform[3]` gets disabled
  - `internaltransform[1]` gets enabled
  - `internaltransform[4]` gets enabled except if the current [map](../../../Enums%20and%20IDs/Maps.md) is `UpperSnekGeizerRoom` where it is disabled
  - The entity `sound` is stopped and placed at the begining of the playback
  - The `Freeze` sound is played at 0.5 volume
- The `actioncooldown` gets set to `vectordata[0].z`, but if the Extra Freeze [medal](../../../Enums%20and%20IDs/Medal.md) is equipped, the value is multiplied by 3 before assigning it

### Kabbu collider logic
- The player entity `hitwall` is set to true
- The `actioncooldown` is set to 0.0
- [GeizerBreak](../GeizerBreak.md) is called
- The `boxcol` is disabled if it is present

### Dropplet cube collider logic
- If the `moveobj` is present:
  - LaunchObject is called with it using a random vector between (-5.0, -10.0, 0.0) and (5.0, 10.0, 0.0) 
  - ServerGeizer is called onm the `moveobj` Hornable which setups the cube to be on the geizer snapped to it
- The RigidBody of the other's parent gets its velocity zeroed out without gravity in kinematic mode
- The grandparent of the other is set to `internaltransform[0]` TODO: huh ???
- `moveobj` is assigned to the other's parent
- The `ingeizer` of the `moveobj`'s Hornable gets set to this `Geizer`

### Player or player follower collider logic
The other gameObject gets rooted to the scene.