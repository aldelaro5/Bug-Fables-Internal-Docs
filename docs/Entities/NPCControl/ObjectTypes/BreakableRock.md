# BreakableRock
A rock that can be broken using Kabbu's Horn Dash or shook by Kabbu's Horn Slash.

## Data Arrays
- `data[0]`: The color of the rock material selectable among 6 different ones:
  - 0: FFFFFF (pure white)
  - 1: ED9121 (mostly orange)
  - 2: EDD872 (light yellow)
  - 3: BCD1ED (light gray)
  - 4: 84BF6B (light green)
  - 5: A03F70 (light magenta)

## Additional data
- `boxcol`: Required to be present with valid data, but the center and size are overriden to (0.0, 2.5, 0.0) and (5.0, 5.0, 5.0) respectively
- `regaionalflag`: If positive, the [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot turned to true when the rock is broken
- `activationflag`: If above 0, the [flag](../../../Flags%20arrays/flags.md) slot turned to true when the rock is broken

## Setup
A few adjustements occurs:
- The entity.`alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity.`rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled

[AddModel](../../EntityControl/Notable%20methods/AddModel.md) is called on the entity with the path `Prefabs/Objects/CrackedRock` with offset (-1.0, 0.0, 1.5). The material's color depends on `data[0]` (any other value has it unchanged):
- 0: FFFFFF (pure white)
- 1: ED9121 (mostly orange)
- 2: EDD872 (light yellow)
- 3: BCD1ED (light gray)
- 4: 84BF6B (light green)
- 5: A03F70 (light magenta)

The tag of this object is set to `DroppletPass` with a layer of 13 (NoDigGround) This was supposed to allow a [Dropplet](Dropplet.md) to pass through the rock instead of colliding with it, but the tag is overriden to `Object` later making this not work.

The following occurs:
- entity.`rotater` tag is set to `Hornable` which allows PlayerControl to get a green ! emoticon when getting 2.5 or lower distance from the grass for 5 frames
- entity.`rigid` has all its constaints frozen
- `boxcol`.center is overriden to (0.0, 2.5, 0.0) and the size is overriden to Vector3.one * 5.0
- [AddPusher](../AddPusher.md) is called and the height of the `pusher` is overriden to 0.0 and the radius to 3.5. The local position of the `pusher` is set to (0.0, 3.0, 0.0).

## Update
If the `timer` hasn't expired yet, it is decremented by the game's frametime clamped from 0.0 to infinity. Otherwise, if it is 0.0 and the entity isn't `dead`, a [Death](../../EntityControl/Notable%20methods/Death.md) coroutine is started with the entity.

## OnTriggerEnter
This only has logic if the `collisionammount` is 1 or below (this is a debounce logic).

If the other gameObject tag is `BeetleDash`, BreakRock is called.

Otherewise, if the other gameObject tag is `BeetleHorn`, ShakeSprite is called on the entity for 10.0 frames using the intensity (0.1, 0.05, 10.0). This will also cause the `Rock2` sound to be played on the entity with a random pitch between 0.9 and 1.1.

`collisionammount` is then set to 2.

## PauseMenu.NearSomething
If the method finds out that an NPCControl of this object type exists less than 4.0 distance from the player, the return will be an array of 3 booleans with the first one being true. This will prevent usage of the Bed Bug [item](../../../Enums%20and%20IDs/Items.md).

## OnCollisionEnter (When this object type is the other collider of a [RollingRock](RollingRock.md))
When a RollingRock receives a collision from a BreakableRock, BreakRock is called on the breakbale one.

## BreakRock
This is a public method that has logic specific to this object type.

If the player is less than 15.0 away from this object and the `RockBreak` sound wasn't playing, it is played at 0.5 volume.

Nothing happens if `hit` is true (prevents to call this when the rock is already broken). If it's false:
- A `Prefabs/Objects/CrackRockBreak` prefab is instantiated at the rock's position and its CrackedRockBreak's initialcolore is set to the rock's material color
- `regionalflag` and `activationflag` flags array slots are set to true when applicable
- `hit` is set to true
- The `boxcol` is disabled
- ShakeScreen is called with the vector being (0.3, 0.3, 0.3) for 0.5 seconds
- [Death](../../EntityControl/Notable%20methods/Death.md#death) is called on the entity with kill
- player.`bounderbreak` is set to 5.0

