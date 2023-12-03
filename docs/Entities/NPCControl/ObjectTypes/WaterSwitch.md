# WaterSwitch
A switch with 2 states to control the vertical position of a child of the map's main mesh GameObject between 2 points (the lower one when not actuated and the upper one when actuated). The child index, the bounds of the y position to set and the amount of frames to spread the movement are configurable.

## Data Arrays
- `data[0]`: A child index of the map's main mesh object that will be moved vertically
- `data[4]`: If it's 1, the switch isn't allowed to be actuated using Kabbu's abilities
- `vectordata[0].x`: The amount of frames to wait between full y position change
- `vectordata[0].y`: The upper bound of the y position of the child pointed by `data[0]`
- `vectordata[0].z`: The lower bound of the y position of the child pointed by `data[0]`

## Additional data
- `activationflag`: If 0 or above, a [flag](../../../Flags%20arrays/flags.md) slot that determines the starting `hit` value and also the slot whose value will be set to this switch `hit` value when toggled.

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
- `internaltransform` is initialised to a single element being the child of the map's main object at index `data[0]` which is the object that will be moved vertically.
- `data[0]` is then overriden to `vectordata[0].x` (the amount of frames to wait between full y position changes)
- `vectordata` is completely overriden to a new array of 4 elements:
    - 0: The loaded `vectordata[0]` before this override
    - 1: The absolute position of `internaltransform[0]`, but the y component is the loaded `vectordata[0].z` (the lower position to have the object move towards)
    - 2: The absolute position of `internaltransform[0]`, but the y component is the loaded `vectordata[0].y` (the higher position to have the object move towards)
    - 3: (The loaded `vectordata[0].x` before this override, 0.0, 0.0)
- `internaltransform[0]`'s isStatic is set to false
- If `hit` is true, `vectordata[0].x` is set to the loaded value (which will have the object move to its upper bound position). Otherwise, `vectordata[0]` is set to Vector3.zero (which will have the object move to its lower bound position).
- The y component of the position of `internaltransform[0]` is set to the loaded `vectordata[0].y` if `hit` is true or to the loaded `vectordata[0].z` if it's false. This sets the starting y position according to the switch `hit` value.

## Update
The y position of `internaltransform[0]` (the object to vertically move) is set via a SmoothLerp from `vectordata[1]` (the lower bound position) to `vectordata[2]` (the higher bound position) with a factor of `vectordata[0].x / vectordata[3].x`. There is a special case where if the `startlife` is less than 20.0 frames, the factor is 1.0 which makes the upper bound be the position to set.

After, this is where `vectordata[0].x` is incremented or decremented by the game's frametime. It is incremented when `hit` is true and when `vectordata[0].x` hasn't reached `vectordata[3].x` yet (meaning it hasn't yet completed its upward travel). It is decremented if `hit` is false and `vectordata[0].x` is above 0.0 (meaning it hasn't yet completed its downward traval). This essentially adjusts the SmoothLerp factor up if `hit` is true and down if `hit` is false as long as it respects the range boundaries from 0.0 to `vectordata[3].x`.

## OnTriggerEnter
Nothing happens if any of the following is true:

- We are in a `pause` or `minipause`
- [message](../../../SetText/Notable%20states.md#message) is grabbed
- `collisionammount` is higher than 1 (this is a debounce protection)
- The other gameObject tag isn't `BeetleHorn`, `BeetleDash`, `Icefall` and `Icecle` while it's not the player `beemerang`
- `data[4]` is not present or it is and its value is 1 while the other gameObject tag is `BeetleHorn` or `BeetleDash` (This basically enforces that `data[4]` being 1 means that only Kabbu's horn can actuate the switch)

The following occurs if we the above is fufilled:

- `collisionammount` is incremented
- A HitPart particle is played at this position + (0.0, 0.5, 0.0)
- `hit` gets toggled
- The `WaterFill` sound is played if `hit` is now true or the `WaterFill2` is played if it is now false. 
- If `activationflag` isn't negative, the corresponding [flag](../../../Flags%20arrays/flags.md) slot is set to `hit`
- `actioncooldown` is then set to 30.0
- If the other gameObject was the player `beemerang`, its `hit` is set to true and the `WoodHit` sound is played on the entity
- If the entity `originalid` isn't -1 (`None`), [SwitchSound](../Notable%20methods/SwitchSound.md#switchsound) is called indicating a press