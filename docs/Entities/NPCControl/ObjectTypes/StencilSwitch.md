# SencilSwitch
A switch that, when activated, causes a frozen area range to appear or to disappear when deactivated.

## Data Arrays
- `data[0]`: ???
- `data[1]`: If this isn't negative, it is an [entity id](../../../SetText/Common%20commands%20id%20schemes/Entity%20id.md) corresponding to this object's parent ??? The parent isn't changed if it's negavtive. NOTE: this must not resolve to a null entity or an exception will be thrown
- `data[2]`: If this is 1, `hit` is set to true, it is left dault otherwise
- `data[3]`: If this is 1, all colliders including the entity's `model` ones are disabled shortly after setup if the model is present, nothing happens otherwise
- `vectordata[0].x`: The multipler of the rate at which the radius grows and shrinks when `hit` is toggled
- `vectordata[0].y`: The diameter of the range
- `vectordata[1]`: The offset to apply to the absolute position if `data[1]` isn't negative

## Setup
First, the entity's `rigid` gets effectively fixed by placing it in kinematic mode without gravity and all constraints frozen. Then, the entity's `alwaysactive` is set to true.

`nointeract` is set to true and if the `activationflag` [flags](../../../Flags%20arrays/flags.md) slot is true, `hit` is also set to true.

From there some adjustements happens based on the `originalid`'s [AnimID](../../../Enums%20and%20IDs/AnimIDs.md)(nothing happens if it doesn't match any of them):
- `SwitchCrystal` and `BigCrystalSwitch`: A GlowTrigger is added to the entity's `model`'s first child with its `parent` set to this object and the `glowparts` set to a single element corresponding to the MeshRender already attached to the first `model` child. 
- `WoodenSwitch` and `SteelSwitch`: `internaldata` is intiialised to a single element being -60 if it's a `WoodenSwitch` or -100 if it's a `SteelSwitch`, Then, the `moveobj` is set to the entity's `model` first child and if `hit` was set to true earlier, its angles are set to (0.0, `internaldata[0]`, 0.0)

After, if the player is present, all collision between the `boxcol` and the player's wall detector are ignored.

Then, if the `originalid` is not among `BigCrystalSwitch`, `WoodenSwitch` or `SteelSwitch`, [AddPushder](../AddPusher.md) is called.

If the entity is allowed to exist according to its `limit`, `requires` and `regionalflag`, then some further setup happens, nothing happens otherwise.

### Enabled StencilSwitch SetUp
First, the entity's `alwaysactive` and `activeonpause` are set to true.

`internaltransform` is initialised to a single element being a new GameObject named `ice radius from ` followed by this object's name with a tag of `IceRadius`. It also gets a child as a new instance of the `Prefabs/Particles/IceRadius` prefab. After that, a trigger SphereCollider is added to it with a radius of 0.8.

`hit` is set to true if `data[1]` is 1.

All collisions between the `boxcol` and the SphereCollider addeed on the `internaltransform[0]` are ignored.

If `data[1]` isn't negative, the following occurs on the entity whose id matches the value:
- this object gets childed to the entity's `model` if it's present or the entity itself if it's not
- this object's absolute position is set to the entity's absolute position + `vectordata[1]`
- This object's entity's `startpos` is set to this transform's absolute position

If `data[3]` is 1 and the entity has a `model`, the collider on that `model` if it exist gets disabled and DisableAllColliders is invoked in 0.1 seconds which disables the entity's `ccol`, the `scol`, the `boxcol` and the `pusher` if any were present.

Finally, `internaltransform[0]` gets childed to the map and its absolute position is set to this object's absolute position.

## Update
First, the scale of `internaltransform[0]` (the ice radius) is set to a lerp from the existing one to Vector3.zero with a factor of the game's frametime * `vectordata[0].x`. If `hit` is true however, the from vector is Vector3.one * `vectordata[0].y` * 2.0 instead. This grows and shrinks the the radius depending on the `hit` value smmoothly.

Then, the position of `internaltransform[0]` (the ice radius) is set to be offscreen at (0.0, -999.0, 0.0) unless `hit` is true which will make the position be set to this object's position instead.

Finally, if `data[1]` isn't negative, the local position of this object is set to `vectordata[1]`.

## OnTriggerEnter
Nothing happens if any of the following is true:
- We are in a `pause` or `minipause`
- [message](../../../SetText/Notable%20states.md#message) is grabbed
- `collisionammount` is higher than 1
- The other gameObject tag isn't `BeetleHorn`, `BeetleDash`, `Icefall` and `Icecle` while it's not the player `beemerang`

The following occurs if we are doing something:
- `collisionammount` is incremented
- A HitPart particle is played at this position + (0.0, 0.5, 0.0)
- All collisions between the entity `ccol` and the SphereCollider of `internaltransform[0]` (the ice radius) are ignored
- The `Freeze` sound is played at this position if `hit` is false, the `IceMelt` one is played if it is true
- `hit` gets toggled
- `actioncooldown` is then set to 30.0
- If `hit` is true, the corresponding [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot of `regionalflag` when it isn't negative is set to true the corresponding [flag](../../../Flags%20arrays/flags.md) slot of `activationflag` when it isn't negative is set to true
- All other `StencilSwitch` in the current map whose entity aren't `iskill` have their `hit` set to false
- If the other gameObject was the player `beemerang`, its `hit` is set to true and the `WoodHit` sound is played on the entity
- If the entity `originalid` isn't -1 (`None`), [SwitchSound](../SwitchSound.md) is called indicating a press
