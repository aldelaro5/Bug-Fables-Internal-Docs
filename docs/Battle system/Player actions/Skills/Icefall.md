# `Icefall`

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Done for each enemy party member that fufills these conditions:<ul><li>Its [position](../../Actors%20states/BattlePosition.md) must not be `OutOfReach` or `Underground`</li><li>After confirming the crosshair position, it must be [IsInRadius](../../Damage%20pipeline/IsInRadius.md) of the enemy party's member with a radius of 3.3 (see the section below for details)</li></ul>|`playerdata[currentturn]`|The enemy party member|`playerdata[currentturn].atk`|[Freeze](../../Damage%20pipeline/AttackProperty.md)|Empty array|false|

## Icicle launch
This action doesn't feature an action command, but it does feature some logic involving an icicle being launch at an angle determined by the player. In a way, it's a "fake" action command because it's logic specific to this action, but it does involve user inputs.

The way it works is after some initial animations, a new UI object called `crosshair` is created rooted to the scene with a sprite of `guisprites[41]` (a crosshair without its ring) on layer 15 (`3DUI`).

From there, all frames are yielded until the Confirm input is pressed, but before the yield, the `crosshair` z angle and x/y position changes. The angle is only for visual effects (it gets incremented by 5x the game's frametime each frame), but the position is what affects the action's logic. The x and y changes with a sin and cos curve respectively (the t term is Time.time):

- x: sin(5t) * 3.25 + 3.5
- y: cos(2x) * 2 + 2.5

This can be summarised by the following:

- The x varies between 0.5 and 6.75 with a period of 2pi / 5 (~1.26 seconds)
- The y varies between 0.5 and 4.5 with a period of pi (~3.1416 seconds)

This essentially creates a sort of zig zag movement over a rectangle with a lower left corner at (0.5, 0.5) and an upper right corner at (6.75, 4.5).

However, it should be noted for the y position, the camera's y `camoffset` will be added to any point if it was above 4.0. This moves the whole rectangle up to line up with the camera and it allows to compensate for a vertically high camera.

The final position when Confirm is pressed will be the one used to call [IsInRadius](../../Damage%20pipeline/IsInRadius.md) with and for damages to be dealt, the point needs to be less than 3.3 units away from the enemy party member. This test is performed for each enemy party members that aren't `OutOfReach` or `Underground`. Damages won't be dealt if the enemy party member is too far away.

This means it allows to target multiple enemy party emmebrs are once, but regardless of where the `crosshair` lands, damages are dealt or not dealt, there is no analogue variation.

## Logic sequence

- Camera is set to look at the center
- attacker animstate set to 105
- `Prefabs/Objects/icecle` instantiated rooted above the party, but with a scale of Vector3.zero and its BoxCollider destroyed
- `Spin` sound played on `sounds[8]` with a pitch of 0.0 on loop
- Over the course of 60.0 frames, the `Prefabs/Objects/icecle`'s scale grows to Vector3.one and its y angle increases continuously until it reaches a 10.0 increment. Also, `sounds[8].pitch` increases towards 1.0
- attacker animstate set to 106
- The `crosshair` UI object is created like mentioned in the section above
- `Crosshair` sound plays on `sounds[9]` with 0.9 pitch and 0.35 volume on loop
- All enemy party members whose [position](../../Actors%20states/BattlePosition.md) isn't `OutOfReach` or `Underground` have [TempCrosshair](../../Visual%20rendering/TempCrosshair.md) called on them without groundpos and the return is stored in a temporary array
- Yield for a frame
- The Confirm input loop mentioned above occurs, but before each yield, `Prefabs/Objects/icecle`'s y angle gets incremented by 10.0. This continues indefinetely until the Confirm input is pressed
- `sounds[9]` is stopped with a 0.1 fade speed
- `sounds[8]` is stopped with a 0.1 fade speed
- `IceMothThrow` sound is played
- attacker animstate set to 107
- `crosshair` is disabled (this is to stop rendering it, but it is still physically present)
- `Prefabs/Objects/icecle` angles are reset to (0.0, 0.0, 45.0 + `crosshair`'s y position * 4.0)
- Over the course of 11/200 frames (~18.18), `Prefabs/Objects/icecle`'s position goes towards the `crosshair`'s position
- All TempCrosshair created earlier gets destroyed
- `IceMothHit` sound plays
- `Prefabs/Particles/mothicenormal` instantiated childed to `Prefabs/Objects/icecle` with -90.0 degrees rotation in x and a scale of (2.0, 2.0, 2.0) and destroyed in 5.0 seconds
- `Prefabs/Objects/icecle` is destroyed
- The position of `crosshair` is obtained and the UI object is then destroyed
- Each enemy is checked for the DoDamage 1 call and it is performed if applicable as mentioned in the section above
- If DoDamage 1 was called at least once, `AtkSucess` sound plays
- Yield for 1.0 seconds
