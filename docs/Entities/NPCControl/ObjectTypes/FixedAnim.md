# FixedAnim
An entity left in a fixed [animstate](../../EntityControl/Animations/animstate.md) ???

## Data Arrays
- `data[0]`: 1 if the `ccol` and the entity's `rigid`'s gravity should be disabled, the `ccol` is remained enabled otherwise with the gravity left unchanged
- `data[1]`: the entity's [animstate](../../EntityControl/Animations/animstate.md) to set

## Setup
Some adjustements occurs on the entity:
- The `ccol` gets disabled or enabled depending on `data[0]`
- The `rigid`'s gravity gets disabled if `data[0]` is 1
- `overridejump` is set to true
- `overrideanim` is set to true
- `animstate` is set to `data[1]`

Finally, the `scol` gets disabled.

## Update
First, the [animstate](../../EntityControl/Animations/animstate.md) is set to `data[1]`. Then, if the Sin of the Time.DeltaTime is 0.8 or above, the entity's `oldstate` is set to -1 TODO: why ???
