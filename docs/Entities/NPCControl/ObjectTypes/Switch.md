TODO: figure out the data and flow better

# Switch
A regular switch that can act as a toggle ???

## Data Arrays
- `data[0]`: ???
- `data[1]`: If it's 1 and the `activationflag` is not negative, the `hit` gets set to the [flag](../../../Flags%20arrays/flags.md) value whose slot is `activationflag`
- `data[2]`: ???
- `data[3]`: ???
- `data[4]`: If it's 1, the entity's `rotater` tag is set to `Hornable`, this is optional, no change happens if it's not 1 or not present. Also, the switch isn't allowed to be actuated using Kabbu's abilities TODO ???

## Setup
First, the entity's `rigid` gets effectively fixed by placing it in kinematic mode without gravity and all constraints frozen. Then, the entity's `alwaysactive` is set to true.

`nointeract` is set to true and if the `activationflag` [flags](../../../Flags%20arrays/flags.md) slot is true, `hit` is also set to true.

From there some adjustements happens based on the `originalid`'s [AnimID](../../../Enums%20and%20IDs/AnimIDs.md)(nothing happens if it doesn't match any of them):
- `SwitchCrystal` and `BigCrystalSwitch`: A GlowTrigger is added to the entity's `model`'s first child with its `parent` set to this object and the `glowparts` set to a single element corresponding to the MeshRender already attached to the first `model` child. 
- `WoodenSwitch` and `SteelSwitch`: `internaldata` is intiialised to a single element being -60 if it's a `WoodenSwitch` or -100 if it's a `SteelSwitch`, Then, the `moveobj` is set to the entity's `model` first child and if `hit` was set to true earlier, its angles are set to (0.0, `internaldata[0]`, 0.0)

After, if the player is present, all collision between the `boxcol` and the player's wall detector are ignored.

Then, if the `originalid` is not among `BigCrystalSwitch`, `WoodenSwitch` or `SteelSwitch`, [AddPushder](../AddPusher.md) is called.

Finally, the entity's `rotater` tag gets set to `Hornable` if `data[4]` is present and 1 and the `hit` gets set to the the [flag](../../../Flags%20arrays/flags.md) value whose slot is `activationflag` if `data[1]` is present and 1 and `activationflag` isn't negative.

## Update
What happens at the start depends on `data[1]` and `data[2]`.

If `data[1]` is 0 and `data[2]` is present and not negative:
- If the `actioncooldown` hasn't expired, it is decremented by the game's frametime. Otherwise, if the `actioncooldown` expired on the last update cycle (checked by being above -1000.0), `hit` is set to false and `actioncooldown` is set to -1100.0 so it doesn't perform this logic on further updates
- If the `activationflag` isn't negative and the [flag](../../../Flags%20arrays/flags.md) slot of it is true, `hit` is set to true.

Otherwise, if `data[1]` is 1 and the `actioncooldown` hasn't expired yet, it is decremented by the game's frametime.

In all cases, if the entity's `originalid` is the `WoodenSwitch` or `SteelSwitch` [AnimID](../../../Enums%20and%20IDs/AnimIDs.md), the `moveobj`'s y angle is set to a LerpAngle from the existing one to 0.0 if `hit` is false or `internaldata[0]` if it's true with a factor of a 1/10 of the game's frametime.

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
- The main logic section occurs, see the section below for details
- If the other gameObject was the player `beemerang`, its `hit` is set to true and the `WoodHit` sound is played on the entity
- If the entity `originalid` isn't -1 (`None`), [SwitchSound](../SwitchSound.md) is called indicating a press

### Main switch logic
What happens first depends on `data[0]`. 

If it's 1, the main logic only happens if `hit` is false or the `TOG` [modifier](../../EntityControl/Modifiers.md) is enabled:
- If `data[1]` isn't negative, the [event](../../../Enums%20and%20IDs/Events.md) at its id is started which this being the caller. Otherwise, the [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot at `regionalflags` and the [flag](../../../Flags%20arrays/flags.md) slot at `activationflag` are set to true
- `hit` is set to true unless the `TOG` modifier is enabled in which case, the value is toggled instead
- If the entity `originalid` is -1 (`None`), its `iskill` is set to true
- If the entity `originalid` is the `WoodenSwitch` [animid](../../../Enums%20and%20IDs/AnimIDs.md), the `moveobj` angles are set to (0.0, -60.0, 0.0)

If it's 0 then it depends on `data[1]`. If it's not 1, then `hit` is set to true and the [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot at `regionalflags` and the [flag](../../../Flags%20arrays/flags.md) slot at `activationflag` are set to true. If it's 1 and the `actioncooldown` expired:
- `hit` is toggled
- `actioncooldown` is set to 30.0
- If the `activationflag` isn't negative, the corresponding [flag](../../../Flags%20arrays/flags.md) slot is set to `hit`
- For each `Switch` in the map excluding this object, if its `data[0]` is 0, its `data[1]` is 1 and its `activationflag` isn't negative and matches this object's, its `hit` is set to this object `hit`

Finally, no matter which branch applied, if `data[2]` exists and it's negative, `actioncooldown` is set to it.