# ANDGate
A multipurpose condition checker that can be operated in 4 distinct modes involving either [flag](../../../Flags%20arrays/flags.md) or other map entities's `hit` value which results in either a new `hit` value or an [event](../../../Enums%20and%20IDs/Events.md) being started.

## Data Arrays
- `data[0]`: This determines the mode of operation when `data[1]` isn't -1 while it is the last data element
- `data[1]`: If this is -1 and it is the last element, `hit` will set to a value that depends on `activationflag`. Otherwise, this is the first element tested corresponding to the mode determined by `data[0]` until the last data element

## Setup
A few adjustements occurs:
- The entity's `alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity's `rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled

## Update
This is where the logic depending on the mode of operation occurs. There are 4 modes in which this object can operate in:
1. Set `hit` based on the `activationflag` [flag](../../../Flags%20arrays/flags.md) slot
2. Set `hit` to `true` if other `hit` value of other map entities have a desired value (set to false otherwise)
3. Set `hit` to `true` if on other [flags](../../../Flags%20arrays/flags.md) slots have a desired value (set to false otherwise)
4. The same then mode 3, but if all conditions satisfies, `hit` isn't set and instead, an [event](../../../Enums%20and%20IDs/Events.md) whose id is `data[0]` is started and this object destroyed.

When checking a data value, it is possible to invert its checked value by sepcifying its negative version, but this means the value 0 cannot be checked for its inverted version. This means for example that if the value is a [flag](../../../Flags%20arrays/flags.md) slot, it will check that it is false instead of checking for being true, but it is not possible to do this for flag slot 0.

Consult the following table for clarity on each mode (one must apply and it will be the first where the required condition is fufilled):
|Mode|Condition|Value(s) checked|Result|
|----:|--------------------|------------|------| 
|1|`data[1]` is -1 and it is the last element|`activationflag`|`hit` is set to the [flag](../../../Flags%20arrays/flags.md) slot of the value.|
|2|`data[0]` is -2|`data[1]` through the last one|If all [flags](../../../Flags%20arrays/flags.md) slots of each of the values are true, `hit` is set to true. It is set to false otherwise.|
|3|`data[0]` is -1|`data[1]` through the last one|If all the `hit` values of each map entities at the index of each of the values are true, `hit` is set to true. It is set to false otherwise.|
|4|`data[0]` is \>= 0|`data[1]` through the last one|If all the `hit` values of each map entities at the index of each of the values are true, an [event](../../../Enums%20and%20IDs/Events.md) whose id is `data[0]` is started and this object gets destroyed. Otherwise, `hit` is set to false.|
