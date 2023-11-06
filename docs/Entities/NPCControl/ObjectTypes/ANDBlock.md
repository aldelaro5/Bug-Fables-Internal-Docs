# ANDBlock
A multipurpose condition checker that can be operated in 3 distinct modes involving either [flag](../../../Flags%20arrays/flags.md) or other map entities's `hit` value which results in a new `hit` value that can contol the entity's `sprite` local position.

## Data Arrays
- `data[0]`: This determines the mode of operation when `data[1]` isn't -1 while it is the last data element
- `data[1]`: If this is -1 and it is the last element, `hit` will set to a value that depends on `activationflag`. Otherwise, this is the first element tested corresponding to the mode determined by `data[0]` until the last data element
- `vectordata[0]`: The vector the entity's `sprite` local position will tends towards during Update when `hit` is true (the factor of the lerp is in `vectordata[1].x`)
- `vectordata[1].x`: The factor of the lerp to use after setting this object's `hit` during Update
- `vectordata[1].y`: ???
- `vectordata[1].z`: ???
- `vectordata[2]`: The entity's `startscale` and the local scale of the `sprite`. This is optional, neither are changed if the magnitude is 0.1 or below.

## Setup
A few adjustements occurs:
- The entity's `alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity's `rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true

Then, further adjustements occurs on the entity:
- `activeonpause` is set to true
- `lockroater` is set to true
- The angles are set to Vector3.zero
- The `rigid` gets all its constraints frozen
- `startscale` and the `sprite`'s local scale is set to `vectordata[2]` when applicable

Then, the object's layer is set to 8 (Ground).

If the entity's `originalid` the `PrisonGate` or the `PrisonGateLocal`'s [AnimID](../../../Enums%20and%20IDs/AnimIDs.md), `internaltransform` is initialised to a single element being the first child of the `model` and it gets childed to the map.

Finally, the `scol` is disabled.

## Update
This is where the logic depending on the mode of operation occurs. There are 3 modes in which this object can operate in and it decides a new value for `hit`:
1. Set `hit` based on the `activationflag` [flag](../../../Flags%20arrays/flags.md) slot
2. Set `hit` to `true` if other `hit` value of other map entities have a desired value (set to false otherwise)
3. Set `hit` to `true` if on other [flags](../../../Flags%20arrays/flags.md) slots have a desired value (set to false otherwise)

When checking a data value, it is possible to invert its checked value by sepcifying its negative version, but this means the value 0 cannot be checked for its inverted version. This means for example that if the value is a [flag](../../../Flags%20arrays/flags.md) slot, it will check that it is false instead of checking for being true, but it is not possible to do this for flag slot 0.

Consult the following table for clarity on each mode (one must apply and it will be the first where the required condition is fufilled):
|Mode|Condition|Value(s) checked|Result|
|----:|--------------------|------------|------| 
|1|`data[1]` is -1 and it is the last element|`activationflag`|`hit` is set to the [flag](../../../Flags%20arrays/flags.md) slot of the value.|
|2|`data[0]` is -2|`data[1]` through the last one|If all [flags](../../../Flags%20arrays/flags.md) slots of each of the values are true, `hit` is set to true. It is set to false otherwise.|
|3|Neither of the other 2 modes applies|`data[1]` through the last one|If all the `hit` values of each map entities at the index of each of the values are true, `hit` is set to true. It is set to false otherwise.|

After the `hit` value was changed to its new one, the local position of the entity's `sprite` is changed to be a lerp from its existing local position to `vectordata[0]` with `vectordata[1].x` as the factor if `hit` is true. If it is false, it's changed to be a lerp from its existing local position to Vector3.zero with `vectordata[1].x` as the factor.
