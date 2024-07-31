# ScrewSwitch
A crank switch actuated by Vi's Beemerang Toss with an analog actuation that needs to be held for a certain period before being fully actuated. The angles of rotation when actuated, the rate of actuation / deactuation and the maximum value to reach before fully actuating the crank are all configurable.

## Data Arrays
- `vectordata[0].x`: The rate at which `actioncooldown` increments per update cycle when the crank is actuated
- `vectordata[0].y`: The rate at which `actioncooldown` decrements per update cycle when the crank is not actuated
- `vectordata[0].z`: The maximum value of `actioncooldown` allowed where the crank is considered fully actuated when reached
- `vectordata[1]`: The base angle vector to rotate the crank every update cycles that it is actuated

## Additional data
- `activationflag`: If above 0, a [flag](../../../Flags%20arrays/flags.md) slot that if was already true on SetUp, the crank will start with its `hit` set to true

## Setup
- entity.`rigid` gets effectively fixed by placing it in kinematic mode without gravity and all constraints frozen 
- entity.`alwaysactive` is set to true
- `nointeract` is set to true
- if the `activationflag` [flags](../../../Flags%20arrays/flags.md) slot is true, `hit` is also set to true.
- From there some adjustements happens based on the `originalid`'s [AnimID](../../../Enums%20and%20IDs/AnimIDs.md)(nothing happens if it doesn't match any of them):
    - `SwitchCrystal` and `BigCrystalSwitch`: A GlowTrigger is added to the entity.`model`'s first child with its `parent` set to this object and the `glowparts` set to a single element corresponding to the MeshRender already attached to the first entity.`model` child.
    - `WoodenSwitch` and `SteelSwitch`: `internaldata` is initialised to a single element being -60 if it's a `WoodenSwitch` or -100 if it's a `SteelSwitch`, Then, the `moveobj` is set to the entity.`model` first child and if `hit` was set to true earlier, its angles are set to (0.0, `internaldata[0]`, 0.0)
- If the player is present, all collision between the `boxcol` and the player's wall detector are ignored.
- If the `originalid` is not among `BigCrystalSwitch`, `WoodenSwitch` or `SteelSwitch`, [AddPushder](../Notable%20methods/AddPusher.md) is called.

## Update
Updates are disabled if the player doesn't exist.

`hit` is set to true if the player.`beemerang` is present and the distance between it and this object is less than 1.75. Otherwise, it is set to false.

If `hit` is true then `actioncooldown` is incremented by the game's frametime * `vectordata[0].x`, but only if the `actioncooldown` is less than `vectordata[0].z` (meaning the crank isn't fully actuated) otherwise, nothing happens.

If `hit` is false and the `actioncooldown` is above 0.0, it is decremented by the game's frametime * `vectordata[0].y`

From there, the ratio of the switch actuation is calculated by multiplying the game's frametime with (`actioncooldown` / `vectordata[0].z`) which is then clamped from 0.0 to 1.0. (0.0 being no actuation and 1.0 being fully actuated).

The entity.`model` angles are set to the actuation ratio * `vectordata[1]` which rotates it proportionally to the crank actuation.

If the ratio is 0 or lower, the entity.`sound` is set to stop looping. Otherwise, [PlaySound](../../EntityControl/EntityControl%20Methods.md#sounds) is called with the `SpinSwitch0` clip at half volume if the entity.`sound` wasn't playing already. It also sets the clip to loop at 0.5 + the actuation ratio as the pitch.

## OnTriggerEnter
If the other gameObject tag is `BeetleHorn`, `hit` is set to true and `actioncooldown` is incremented by `vectordata[0].x` * 10.0 then clamped from 0.0 to `vectordata[0].z`. This slightly actuate the crank the moment Kabbu's Horn Slash collides with it.
