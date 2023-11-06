# BreakableRock
A rock that can be broken using Kabbu's Horn Dash ???

## Data Arrays
- `data[0]`: ???

## Setup
A few adjustements occurs:
- The entity's `alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity's `rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled

Then, AddModel is called on the entity with the path `Prefabs/Objects/CrackedRock` with offset (-1.0, 0.0, 1.5). The material's color depends on `data[0]` (any other value has it unchanged):
- 0: FFFFFF (pure white)
- 1: ED9121 (mostly orange)
- 2: EDD872 (light yellow)
- 3: BCD1ED (light gray)
- 4: 84BF6B (light green)
- 5: A03F70 (light magenta)

Then, the tag of this object is set to `DroppletPass` with a layer of 13 (NoDigGround). The entity's `rotater` tag is set to `Hornable`. The entity's `rigid` has all its constaints frozen.

Then, the `boxcol`'s center is overriden to (0.0, 2.5, 0.0) and the size is overriden to Vector3.one * 5.0.

Finally, [AddPusher](../AddPusher.md) is called and the height of the `pusher` is overriden to 0.0 and the radius to 3.5. The local position of the `pusher` is set to (0.0, 3.0, 0.0).

## Update
If the `timer` hasn't expired yet, it is decremented by the game's frametime clamped from 0.0 to infinity. Otherwise, if it is 0.0 and the entity isn't `dead`, a [Death](../../EntityControl/Notable%20methods/Death.md) coroutine is started with the entity.

## OnTriggerEnter
This only has logic if the `collisionammount` is 1 or below.

If the other gameObject tag is `BeetleDash`, [BreakRock](../BreakRock.md) is called.

Otherewise, if the other gameObject tag is `BeetleHorn`, ShakeSprite is called on the entity for 10.0 frames using the intendity (0.1, 0.05, 10.0). This will also cause the `Rock2` sound to be played on the entity with a random pitch between 0.9 and 1.1.

Finally, `collisionammount` is set to 2.
