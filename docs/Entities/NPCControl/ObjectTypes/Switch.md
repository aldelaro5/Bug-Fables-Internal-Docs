# Switch
A switch that can be turned on or off offering a wide variety of actuation flow such as starting an [event](../../../Enums%20and%20IDs/Events.md), changing other `hit` values, offer a toggling feature and be only actuable using Kabbu's horn. Consult the switch actuation section for more details.

## Data Arrays
- `data[0]`: Controls the actuation flow, see the section below for details
- `data[1]`: Controls the actuation flow, see the section below for details
- `data[2]`: If `data[1]` is 0, this the the amount of frames that must pass after an actuation for the switch to automatically turn itself off. This is optional, this feature isn't used when it doesn't exist or is negative. As a side effect of enabling this feature, the `hit` value is continuously set to true if the `activationflag` [flag](../../../Flags%20arrays/flags.md) slot is true
- `data[3]`: UNUSED (The value is used in an if statement in Update, but the if leads to no code)
- `data[4]`: If it's 1, the entity's `rotater` tag is set to `Hornable` (allows PlayerControl to get a green ! emoticon when getting 2.5 or lower distance from the switch for 5 frames) and disallows the switch to be actuated using Leif's Icicle or Vi's [beemerang](Beemerang.md). This is optional, no change happens if it's 0 or not present. NOTE: anything other than 0 or 1 is considered invalid and will prevent the switch from being actuated under any circumstances

## Additional data
- `boxcol`: Required to be present with valid data
- `regaionalflag`: If positive, a [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot, see the table below for details
- `activationflag`: If above 0, a [flag](../../../Flags%20arrays/flags.md) slot, see the table below for details

## About switch actuation
The meaning of `data[0]` and `data[1]` these 2 data elements have very complex semantics. They together control the flow of the actuation of the switch which is stored in the `hit` value as well as how the `activationflag` and `regionalflag` slots are used. 

`data[1]` being 0 allows `data[2]` to function if it exists and is positive which the table will mention when it is applicable. The implications of this when applicable are outlined in the table below.

To note, `data[4]` acts completely independently of `data[0]` and `data[1]`. It only permits actuation from Kabbu's horn and makes the switch `Hornable` which allows PlayerControl to get a green ! emoticon when getting 2.5 or lower distance from the switch for 5 frames.

Unless stated otheriwse, `activationflag` and `regionalflag` only applies when the former is above 0 and when the latter is positive. Here is a breakdown of all actuation flow:

|`data[0]`|`data[1]`|Starting `hit` value|Actuation flow|Effects on actuation|
|---------|---------|-------------------|--------------|---------------|
|0|0|If the `activationflag` flag slot (with the 0 value being allowed) is true, the switch will continuously set its `hit` to true during Update if `data[2]` exists and isn't negative.|This switch can only be toggled on. If `data[2]` exists and is positive, the switch automatically turns itself off after the amount of frames contained in `data[2]` passed. Otherwise, the switch cannot be turned off|When `hit` turns true, both `activationflag` and `regionalflag` respective slots are set to true|
|0|1|`activationflag` (with the 0 value allowed) tells the starting value of `hit` by its flag slot.|The switch can be toggled on and off. Once toggled, there is a 30 frames cooldown before being able to toggle it again, but `data[2]` can override the cooldown time if it exist and isn't negative|When actuated, the `activationflag` flag slot is set to the new `hit` value. Additionally, all other switch entities present with the same `data[0]`, `data[1]` and `activationflag` values will have their `hit` values set to the new one after the toggle.<sup>1</sup>|
|1|Negative|false|The switch can only be turned on once unless the `TOG` [modifier](../../EntityControl/Modifiers.md) is active which allows the switch to be toggled on and off.|When the switch is actuated, both `activationflag` and `regionalflag` respective slots are set to true. Additionally, if entity.`originalid` has an undefined (-1) [animid](../../../Enums%20and%20IDs/AnimIDs.md), entity.`iskill` is set to true and if it's `WoodenSwitch`, the `moveobj` angles are set to (0.0, -60.0, 0.0) (This moves the lever stick)|
|1|0 or above|false<sup>2</sup>|The same actuation flow than if `data[1]` was negative<sup>3</sup>|That same effects occurs than if `data[1]` was negative with the addition that `data[1]` is an [event](../../../Enums%20and%20IDs/Events.md) id that will trigger with this switch as the caller each time the switch is actuated|

1: This combination is practically unused under normal gameplay as it is likely an older implementation of [event](../../../Enums%20and%20IDs/Events.md) 169

2: If `data[1]` is 1, the starting value of `hit` is the value of the `activationflag` (0 included) flag slot

3: If `data[1]` is 0, it allows `data[2]` if it exists and isn't negative to make the switch automatically turns itself off after the amount of frames contained in `data[2]` passed after each actuation. It also sets the `hit` value to true continuously during Update if `activationflag` (0 included) flag slot is true

## Setup
- entity.`rigid` gets effectively fixed by placing it in kinematic mode without gravity and all constraints frozen 
- entity.`alwaysactive` is set to true
- `nointeract` is set to true
- if the `activationflag` [flags](../../../Flags%20arrays/flags.md) slot is true, `hit` is also set to true.
- From there some adjustements happens based on the `originalid`'s [AnimID](../../../Enums%20and%20IDs/AnimIDs.md)(nothing happens if it doesn't match any of them):
  - `SwitchCrystal` and `BigCrystalSwitch`: A GlowTrigger is added to the entity's `model`'s first child with its `parent` set to this object and the `glowparts` set to a single element corresponding to the MeshRender already attached to the first `model` child.
  - `WoodenSwitch` and `SteelSwitch`: `internaldata` is initialised to a single element being -60 if it's a `WoodenSwitch` or -100 if it's a `SteelSwitch`, Then, the `moveobj` is set to the entity's `model` first child and if `hit` was set to true earlier, its angles are set to (0.0, `internaldata[0]`, 0.0)
- If the player is present, all collision between the `boxcol` and the player's wall detector are ignored.
- If the `originalid` is not among `BigCrystalSwitch`, `WoodenSwitch` or `SteelSwitch`, [AddPushder](../AddPusher.md) is called.
- If `data[4]` is 1 the entity's `rotater` tag gets set to `Hornable` (allows PlayerControl to get a green ! emoticon when getting 2.5 or lower distance from the switch for 5 frames).

## Update
What happens at the start depends on `data[1]` and `data[2]`.

If `data[1]` is 0 and `data[2]` is present and not negative:
- If the `actioncooldown` hasn't expired, it is decremented by the game's frametime. Otherwise, if the `actioncooldown` expired on the last update cycle (checked by being above -1000.0), `hit` is set to false and `actioncooldown` is set to -1100.0 so it doesn't perform this logic on further updates (this is the autooff feature taking effect)
- If the `activationflag` isn't negative and the [flag](../../../Flags%20arrays/flags.md) slot of it is true, `hit` is set to true.

Otherwise, if `data[1]` is 1 and the `actioncooldown` hasn't expired yet, it is decremented by the game's frametime.

In all cases, if the entity.`originalid` is the `WoodenSwitch` or `SteelSwitch` [AnimID](../../../Enums%20and%20IDs/AnimIDs.md), the `moveobj`'s y angle is set to a LerpAngle from the existing one to 0.0 if `hit` is false or `internaldata[0]` (-60.0 for a `WoodenSwitch` or -100.0 for a `SteelSwitch`) if it's true with a factor of a 1/10 of the game's frametime.

## OnTriggerEnter
Nothing happens if any of the following is true:
- We are in a `pause` or `minipause`
- [message](../../../SetText/Notable%20states.md#message) is grabbed
- `collisionammount` is higher than 1 (this is a debounce protection)
- The other gameObject tag isn't `BeetleHorn`, `BeetleDash`, `Icefall` and `Icecle` while it's not the player `beemerang`
- `data[4]` is not present or it is and its value is 1 while the other gameObject tag is `BeetleHorn` or `BeetleDash` (This basically enforces that `data[4]` being 1 means that only Kabbu's horn can actuate the switch)

The following occurs:
- `collisionammount` is incremented
- A HitPart particle is played at this position + (0.0, 0.5, 0.0)
- The main logic section occurs, see the section below for details
- If the other gameObject was the player `beemerang`, its `hit` is set to true and the `WoodHit` sound is played on the entity
- If the entity `originalid` isn't -1 (`None`), [SwitchSound](../SwitchSound.md) is called indicating a press

What happens first depends on `data[0]`. 

### `data[0]` is 1
This does nothing if `hit` is true and the `TOG` [modifier](../../EntityControl/Modifiers.md) is not active:
- If `data[1]` isn't negative, the [event](../../../Enums%20and%20IDs/Events.md) at its id is started which this being the caller. Otherwise, the [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot at `regionalflags` and the [flag](../../../Flags%20arrays/flags.md) slot at `activationflag` are set to true
- `hit` is set to true unless the `TOG` [modifier](../../EntityControl/Modifiers.md) is active in which case, the value is toggled instead
- If the entity `originalid` is -1 (`None`), its `iskill` is set to true
- If the entity `originalid` is the `WoodenSwitch` [animid](../../../Enums%20and%20IDs/AnimIDs.md), the `moveobj` angles are set to (0.0, -60.0, 0.0)

If it's 0 then it depends on `data[1]`. If it's not 1, then `hit` is set to true and the [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot at `regionalflags` and the [flag](../../../Flags%20arrays/flags.md) slot at `activationflag` are set to true. If it's 1 and the `actioncooldown` expired:
- `hit` is toggled
- `actioncooldown` is set to 30.0
- If the `activationflag` isn't negative, the corresponding [flag](../../../Flags%20arrays/flags.md) slot is set to `hit`
- For each `Switch` in the map excluding this object, if its `data[0]` is 0, its `data[1]` is 1 and its `activationflag` isn't negative and matches this object's, its `hit` is set to this object `hit`

### Common end
Finally, no matter which branch applied, if `data[2]` exists and it's negative, `actioncooldown` is set to it.

## Hazards.OnObjectStay
If the NPCControl passed has this type, it returns true which allows it to stay in collision with the Hazards.