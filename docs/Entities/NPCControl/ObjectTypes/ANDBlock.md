# ANDBlock
A multipurpose condition checker that can be operated in 3 distinct modes involving either [flags](../../../Flags%20arrays/flags.md) or other NPCControl's `hit` value which results in a new `hit` value that can contol the entity.`sprite` local position.

## Data Arrays
- `data`: Control the mode of operations and data to check. See the table below for details. 
- `vectordata[0]`: The local position to set the entity.`sprite` when `hit` goes to true.
- `vectordata[1].x`: The factor of the lerp to use after setting this object's `hit` during Update
- `vectordata[2]`: The entity.`startscale` and the local scale of the entity.`sprite`. This is optional, neither are changed if the magnitude is <= 0.1 or does not exist.

## Additional data
- `activationflag`: When configured as such, the `hit` value is set to the value of this [flag](../../../Flags%20arrays/flags.md) slot. See the table below for details.

## Setup
A few adjustements occurs:

- The entity.`alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity.`rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true
- entity.`activeonpause` is set to true
- entity.`lockroater` is set to true
- The angles of the entity are set to Vector3.zero
- The entity.`rigid` gets all its constraints frozen
- entity.`startscale` and the entity.`sprite` local scale is set to `vectordata[2]` when applicable
- The object's layer is set to 8 (Ground)
- The `boxcol` meterial is set to `defaultpmat` if it exists
- The `scol` is disabled

If the entity.`originalid` is the `PrisonGate` or the `PrisonGateLocal`'s [AnimID](../../../Enums%20and%20IDs/AnimIDs.md), `internaltransform` is initialised to a single element being the second child of the `model` and it gets childed to the map (this is the base of the gate at the bottom). The reason this is done is because when the local position changes, only the grate part of the model needs to move, but the base still needs to render while being unaffected by the grate movement.

## Update
This is where the logic depending on the mode of operation occurs. There are 3 modes in which this object can operate in and it decides a new value for `hit`:

1. Set `hit` based on the `activationflag` [flag](../../../Flags%20arrays/flags.md) slot
2. Set `hit` to `true` if other [flags](../../../Flags%20arrays/flags.md) slots have a desired value (set to false otherwise)
3. Set `hit` to `true` if other `hit` value of other NPCControl have a desired value (set to false otherwise)

When checking a data value, it is possible to invert its checked value by sepcifying its negative version, but this means the value 0 cannot be checked for its inverted version. This means for example that if the value is a [flag](../../../Flags%20arrays/flags.md) slot, it will check that it is false instead of checking for being true, but it is not possible to do this for flag slot 0.

Consult the following table for clarity on each mode (one must apply and it will be the first where the required condition is fufilled):

|Mode|Condition|Value(s) checked|Result|
|----:|--------------------|------------|------| 
|1|`data[1]` is -1 and it is the last element|`activationflag`|`hit` is set to the [flag](../../../Flags%20arrays/flags.md) slot of the value.|
|2|`data[0]` is -2|`data[1]` through the last one|If all [flags](../../../Flags%20arrays/flags.md) slots of each of the values are true, `hit` is set to true. It is set to false otherwise.|
|3|Neither of the other 2 modes applies|`data[1]` through the last one|If all the `hit` values of each NPCControl at the index of each of the values are true, `hit` is set to true. It is set to false otherwise.|

After the `hit` value is changed to its new one, the local position of the entity.`sprite` is changed to be a lerp from its existing local position to `vectordata[0]` with `vectordata[1].x` as the factor if `hit` is true. If it is false, it's changed to be a lerp from its existing local position to Vector3.zero with `vectordata[1].x` as the factor.
