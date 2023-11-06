# BeetleGrass
A grass that can be cut by Kabbu's Horn Slash ???

## Data Arrays
- `data[0]`: The id of the grass sprite to use (NOTE: any other value than the ones below when `data[1]` doesn't apply will cause an exception to be thrown):
  - 0: a standard green grass
  - 1: a branch looking grass
  - 2: an orange grass
  - 3: a brown grass
  - 4: a dense dark green grass
  - 5: a purple grass
  - 6: a beige and gray grass
- `data[1]`: The [crystalbfflags](../../../Enums%20and%20IDs/crystalbfflags.md) id if applicable (being negative or not present means it isn't)

## HasHiddenItem conditions
If `data[1]` is present and not negative, the [crystalbflags](../../Enums%20and%20IDs/crystalbfflags.md) whose id is that value must not have been obtained yet.

## Setup
A few adjustements occurs:
- The entity's `alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity's `rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled

Then, if `data[1]` is present and not negative and the crystal berry corresponding to its id has been obtained, the entity's `iskill` is set to true which ends this object setup as the grass will not appear.

Otherwise, the layer is set to 8 (Ground) and the `boxcol`'s material is set to the defaultpmat. Finally, adjustements on the entity are performed:
- The `rotater`'s tag is set to `Hornable`
- The `rigid` has all constraints frozen
- `overrideanim` is set to true
- The `sprite` is enabled and set to the corresponding sprite from grasssprite mentioned above and its shadowCastingMode is set to TwoSided
- Unless nowindeffect is turned on, the `sprite`'s material is set to the windShader and RefreshWind is called on the `sprite` ???

## Update
If the `timer` hasn't expired yet, it is decremented by the game's frametime clamped from 0.0 to infinity. Otherwise, if it is 0.0 and the entity isn't `dead`, a [Death](../../EntityControl/Notable%20methods/Death.md) coroutine is started with the entity.

## OnTriggerEnter
If the other gameObject tag is `BeetleHorn` or `BeetleDash` while `hit` is false and the distance between this and the other transform is less than 3.5, CutGrass is called followed by a RefreshPlayer call. TODO: why do we need to call RefreshPlayer ???

## CutGrass
TODO