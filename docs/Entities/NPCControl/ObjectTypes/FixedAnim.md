# FixedAnim
An object where the entity is left in a fixed [animstate](../../EntityControl/Animations/animstate.md) the the option to also disables the `ccol` and gravity on it.

## Data Arrays
- `data[0]`: If 1, the entity.`ccol` and the entity.`rigid`'s gravity are disabled
- `data[1]`: the entity's [animstate](../../EntityControl/Animations/animstate.md) to set

## Setup
- The entity.`ccol` gets disabled or enabled depending on `data[0]`
- The entity.`rigid`'s gravity gets disabled if `data[0]` is 1
- entity.`overridejump` is set to true
- entity.`overrideanim` is set to true
- entity.[animstate](../../EntityControl/Animations/animstate.md) is set to `data[1]`
- The `scol` gets disabled.

## Update
The [animstate](../../EntityControl/Animations/animstate.md) is set to `data[1]`. Then, if the Sin of the Time.DeltaTime is 0.8 or above (Requires it to at least 0.9273), the entity's `oldstate` is set to which forces [UpdateSprite](../../EntityControl/Update%20process/UpdateSprite.md) to run its main logic. NOTE: it is unknown why the Sin is there, but it doesn't seem to have an impact under normal gameplay.
