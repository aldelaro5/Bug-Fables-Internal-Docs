# SencilSwitch
A switch that, when activated, causes a frozen area range to appear or to disappear when deactivated.

## Data Arrays
- `data[1]`: If this isn't negative, it is a map entity id whose entity is set as this stencil switch's parent. No parent changes occurs otherwise
- `data[2]`: If this is 1, `hit` is set to true on SetUp regardless of the `activationflag`
- `data[3]`: If this is 1, all colliders including the entity.`model` ones are disabled shortly after setup if the model is present, nothing happens otherwise
- `vectordata[0].x`: The the rate at which the radius grows and shrinks when `hit` is toggled (1.0 being the game's frametime)
- `vectordata[0].y`: The radius of the range
- `vectordata[1]`: The local position of the stencil switch if `data[1]` isn't negative

## Additional data
- `activationflag`: If 0 or above, a [flag](../../../Flags%20arrays/flags.md) slot that determines the starting `hit` value and also the slot whose value will be set to this switch `hit` value when toggled.
- `regaionalflag`: If not negative, the [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot turned to true when the stencil switch is actuated for the first time

## Setup
- entity.`rigid` gets effectively fixed by placing it in kinematic mode without gravity and all constraints frozen 
- entity.`alwaysactive` is set to true
- `nointeract` is set to true
- if the `activationflag` [flags](../../../Flags%20arrays/flags.md) slot is true, `hit` is also set to true.
- From there some adjustements happens based on the `originalid`'s [AnimID](../../../Enums%20and%20IDs/AnimIDs.md)(nothing happens if it doesn't match any of them):
  - `SwitchCrystal` and `BigCrystalSwitch`: A GlowTrigger is added to the entity's `model`'s first child with its `parent` set to this object and the `glowparts` set to a single element corresponding to the MeshRender already attached to the first `model` child.
  - `WoodenSwitch` and `SteelSwitch`: `internaldata` is initialised to a single element being -60 if it's a `WoodenSwitch` or -100 if it's a `SteelSwitch`, Then, the `moveobj` is set to the entity's `model` first child and if `hit` was set to true earlier, its angles are set to (0.0, `internaldata[0]`, 0.0)
- If the player is present, all collision between the `boxcol` and the player's wall detector are ignored.
- If the `originalid` is not among `BigCrystalSwitch`, `WoodenSwitch` or `SteelSwitch`, [AddPushder](../Notable%20methods/AddPusher.md) is called.

If the entity is allowed to exist according to its `limit`, `requires` and `regionalflag`, then some further setup happens, nothing happens otherwise.

### Enabled StencilSwitch SetUp
- entity.`alwaysactive` is set to true
- entity.`activeonpause` is set to true
- `internaltransform` is initialised to a single element being a new GameObject named `ice radius from ` followed by this entity name with a tag of `IceRadius`. It gets a child added as a new instance of the `Prefabs/Particles/IceRadius` prefab. After that, a trigger SphereCollider is added to it with a radius of 0.8.
- `hit` is set to true if `data[1]` is 1.
- All collisions between the `boxcol` and the SphereCollider addeed on the `internaltransform[0]` are ignored.
- If `data[1]` isn't negative, the map entity whose id is this value is resolved and the following occurs:
  - This stencil switch gets childed to the resolved entity.`model` if it's present or the entity itself if it's not
  - This stencil switch position is set to the resolved entity position + `vectordata[1]`
  - This stencil switch entity.`startpos` is set to its position
- If `data[3]` is 1 and the entity has a `model`, the collider on that `model` if it exist gets disabled and DisableAllColliders is invoked in 0.1 seconds which disables the entity.`ccol`, the `scol`, the `boxcol` and the `pusher` if any were present
- `internaltransform[0]` gets childed to the map and its position is set to this stencil switch position

## Update
The scale of `internaltransform[0]` (the ice radius) is set to a lerp from the existing one to a value that depends on the `hit` value of this stencil switch. If it's false, it's Vector3.zero and if it's true, it's Vector3.one * `vectordata[0].y` * 2.0 instead. The factor is the game's frametime * `vectordata[0].x`. This grows and shrinks the the radius depending on the `hit` value smmoothly.

The position of `internaltransform[0]` (the ice radius) is set to be offscreen at (0.0, -999.0, 0.0) unless `hit` is true which will make the position be set to this object's position instead.

Finally, if `data[1]` isn't negative (another map entity is the parent of this stencil switch), the local position of this object is set to `vectordata[1]`.

## MapControl.CheckStencilSwitch
This method specifically targets any map entities of this object type. It is called for any MapControl LateUpdate after the first one.

If any exists in the map where their entity isn't `iskill` and their `hit` value is true, it will select the first one. With it, 2 shaders global properties are set:
- `GlobalIceRadius`: Half of the magnitude of the `internfaltransform[0]` (the ice radius object) scale
- `CentralIcePoint`: the sencil switch position

It also sets the map's `stencilid` to the map entity id of the selected stencil switch which has the effect of freezing any IcePlatform in range of the stencil switch which is defined as being less than `vectordata[0].y` * 1.85 away from the stencil switch.

If no actuated stencil switches are found, the `stencilid` remains at -1 and the `GlobalIceRadius` shader property is set to lerp from the existing one to 0.0 with a factor of 1/20 of the game's frametime.

## OnTriggerEnter
Nothing happens if any of the following is true:
- We are in a `pause` or `minipause`
- [message](../../../SetText/Notable%20states.md#message) is grabbed
- `collisionammount` is higher than 1 (this is a debounce protection)
- The other gameObject tag isn't `BeetleHorn`, `BeetleDash`, `Icefall` and `Icecle` while it's not the player `beemerang`

The following occurs if we the above is fufilled:
- `collisionammount` is incremented
- A HitPart particle is played at this position + (0.0, 0.5, 0.0)
- All collisions between the entity.`ccol` and the SphereCollider of `internaltransform[0]` (the ice radius) are ignored
- The `Freeze` sound is played at this position if `hit` is false, the `IceMelt` one is played if it is true
- `hit` gets toggled
- `actioncooldown` is set to 30.0 (doesn't do anything)
- If `hit` is true, the corresponding [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot of `regionalflag` when it isn't negative is set to true the corresponding [flag](../../../Flags%20arrays/flags.md) slot of `activationflag` when it isn't negative is set to true
- DeactivateOtherStencil is called which deactivates all other `StencilSwitch` in the current map whose entity aren't `iskill` by having their `hit` set to false
- If the other gameObject was the player `beemerang`, its `hit` is set to true and the `WoodHit` sound is played on the entity
- If the entity `originalid` isn't -1 (`None`), [SwitchSound](../SwitchSound.md) is called indicating a press

## EntityControl.OnTriggerStay
Due to the `IceRadius` tag of the ice radius object, any entities that collides with it has their `inice` set to true which update thier sprites and animation accordingly.