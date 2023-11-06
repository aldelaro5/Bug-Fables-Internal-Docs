# WaterSwitch
A switch with 2 states to control the vertical position of a child of the map's main GameObject between 2 points. The child index, the bounds of the y position to set and the amount of frames to spread the movevement are configurable.

## Data Arrays
- `data[0]`: A child index of the map's main object that will be moved vertically.
- `data[1]`: ???
- `data[2]`: ???
- `data[3]`: ???
- `data[4]`: If it's 1, the switch isn't allowed to be actuated using Kabbu's abilities
- `vectordata[0].x`: The amount of frames to wait between full y position change.
- `vectordata[0].y`: The upper bound of the y position of the child pointed by `data[0]`
- `vectordata[0].z`: The lower bound of the y position of the child pointed by `data[0]`

## Setup
First, the entity's `rigid` gets effectively fixed by placing it in kinematic mode without gravity and all constraints frozen. Then, the entity's `alwaysactive` is set to true.

`nointeract` is set to true and if the `activationflag` [flags](../../../Flags%20arrays/flags.md) slot is true, `hit` is also set to true.

From there some adjustements happens based on the `originalid`'s [AnimID](../../../Enums%20and%20IDs/AnimIDs.md)(nothing happens if it doesn't match any of them):
- `SwitchCrystal` and `BigCrystalSwitch`: A GlowTrigger is added to the entity's `model`'s first child with its `parent` set to this object and the `glowparts` set to a single element corresponding to the MeshRender already attached to the first `model` child. 
- `WoodenSwitch` and `SteelSwitch`: `internaldata` is intiialised to a single element being -60 if it's a `WoodenSwitch` or -100 if it's a `SteelSwitch`, Then, the `moveobj` is set to the entity's `model` first child and if `hit` was set to true earlier, its angles are set to (0.0, `internaldata[0]`, 0.0)

After, if the player is present, all collision between the `boxcol` and the player's wall detector are ignored.

Then, if the `originalid` is not among `BigCrystalSwitch`, `WoodenSwitch` or `SteelSwitch`, [AddPushder](../AddPusher.md) is called.

Then, `internaltransform` is initialised to a single element being the child of the map's main object at index `data[0]`.

`data[0]` is then overriden to `vectordata[0].x` and `vectordata` is completely overriden to a new array of 4 elements:
- 0: The loaded `vectordata[0]`
- 1: The absolute position of `internaltransform[0]`, but the y component is the loaded `vectordata[0].z`
- 2: The absolute position of `internaltransform[0]`, but the y component is the loaded `vectordata[0].y`
- 3: (The loaded `vectordata[0].x`, 0.0, 0.0)

`internaltransform[0]`'s isStatic is set to false.

If `hit` is true, `vectordata[0].x` is set to the loaded value. Otherwise, `vectordata[0]` is set to Vector3.zero.

Finally, the y component of the absolute position of `internaltransform[0]` is set to the loaded `vectordata[0].y` if `hit` is true or to the loaded `vectordata[0].z` if it's false.

## Update
First, the y position of `internaltransform[0]` is set via a SmoothLerp from `vectordata[1]` to `vectordata[2]` with a factor of `vectordata[0].x / vectordata[3].x`. There is a special case where if the `startlife` is less than 20.0 frames, the factor is 1.0 which makes the upper bound be the position to set.

After, this is where `vectordata[0].x` is incremented or decremented by the game's frametime. It is incremented when `hit` is true and when `vectordata[0].x` hasn't reached `vectordata[3].x` yet (if it has, nothing happens). It is decremented if `hit` is false and `vectordata[0].x` is above 0.0 (nothing happens if it is 0.0 or below). This essentially adjusts the SmoothLerp factor up if `hit` is true and down if `hit` is false as long as it respects the boundaries from 0.0 to `vectordata[3].x`.

## OnTriggerEnter
Nothing happens if any of the following is true:
- We are in a `pause` or `minipause`
- [message](../../../SetText/Notable%20states.md#message) is grabbed
- `collisionammount` is higher than 1
- The other gameObject tag isn't `BeetleHorn`, `BeetleDash`, `Icefall` and `Icecle` while it's not the player `beemerang`
- `data[4]` is not present or it is and its value is 1 while the other gameObject tag is `BeetleHorn` or `BeetleDash`

The following occurs if we are doing something:
- `collisionammount` is incremented
- A HitPart particle is played at this position + (0.0, 0.5, 0.0)
- `hit` gets toggled
- The `WaterFill` sound is played if `hit` is now true or the `WaterFill2` is played if it is now false. 
- If `activationflag` isn't negative, the corresponding [flag](../../../Flags%20arrays/flags.md) slot is set to `hit`
- `actioncooldown` is then set to 30.0
- If the other gameObject was the player `beemerang`, its `hit` is set to true and the `WoodHit` sound is played on the entity
- If the entity `originalid` isn't -1 (`None`), [SwitchSound](../SwitchSound.md) is called indicating a press