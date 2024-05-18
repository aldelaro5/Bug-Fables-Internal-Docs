# `NeedleToss`

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Done on the first enemy party member (checked in `enemydata` order every 2 frames starting from the second one up to 41.0 frames) that fufills these conditions:<ul><li>Its [position](../../Actors%20states/BattlePosition.md) must not be `Underground`</li><li>Its `hp` must be above 0</li><li>After confirming the crosshair position, its world [CenterPos](../../Actors%20states/CenterPos.md) + 0.25 in x must be within a radius of 1.5 of the thrown dart (see the target selection section for more details)</li></ul>|`playerdata[currentturn]`|The enemy party member|`playerdata[currentturn].atk` - 1 unclamped|[Pierce](../../Damage%20pipeline/AttackProperty.md)|Empty array|false|
|2|Same as DoDamage 1, but this is the second throw only|`playerdata[currentturn]`|The enemy party member|Depends if this is the same enemy party member targetted by DoDamage 1:<ul><li>If it's the same, it's `playerdata[currentturn].atk` - 1 * 0.65 floored clamped from 1 to `playerdata[currentturn].atk` - 1. NOTE: This clamp breaks with any base `atk` values of 1 and below because the lower bound is higher than the upper bound. The lower bound clamps always takes priority over the upper bound clamp in these cases</li><li>If it's not the same, it's `playerdata[currentturn].atk` - 1 unclamped</li></ul>|[Pierce](../../Damage%20pipeline/AttackProperty.md)|Empty array|false|
|3|Same as DoDamage 1 and 2, but this is the third throw only which can only happens if the `Beemerang2` [medal](../../../Enums%20and%20IDs/Medal.md) was equipped|`playerdata[currentturn]`|The enemy party member|Depends if this is the same enemy party member targetted by DoDamage 2:<ul><li>If it's the same, it's `playerdata[currentturn].atk` - 1 * 0.65 floored * 0.65 floored clamped from 1 to `playerdata[currentturn].atk` - 1. NOTE: This clamp breaks with any base `atk` values of 1 and below because the lower bound is higher than the upper bound. The lower bound clamps always takes priority over the upper bound clamp in these cases</li><li>If it's not the same, it's `playerdata[currentturn].atk` - 1 unclamped</li></ul>|[Pierce](../../Damage%20pipeline/AttackProperty.md)|Empty array|false|

## Target selection
This action features custom target selection logic involving a  "fake" action command where a cursor periodically rotate along the attacker which dictates the direction of a dart's throwing direction.

How it works is there will be 3 sprites setup in a line formation around the attacker that will rotate periodically. The last 2 are hexagons that are only visually align with the crosshair and the attacker which the first one is the crosshair and is the furthest away. The crosshair is the one that matters to determine the direction.

For the direction, there's an infinite input loop that won't stop until the Confirm input is pressed. This will be done for each thrown darts which is normally 2, but can be 3 if the `Beemerang2` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped. Here's what happens during this loop on each frame:

- The crosshair position is set to a new postion where the x and y are components are the result of sin and cos curve:
    - x: attacker's x position (should be -3.5 by then) + 1.0 + abs(cos(Time.time * 2.5)) * 2.0
    - y: 3.5 + sin(Time.time * 2.5) * 2.55
    - z: 0.0
- The 2 hexagons rotates by -6.0 in z
- The closest hexagon's position is set to a lerp from the existing one + 3.5 in y to the crosshair with a factor of 1.0/3.0
- The closest hexagon's position is set to a lerp from the existing one + 3.5 in y to the crosshair with a factor of 2.0/3.0

This means the crosshair positions varies in the following manner:

- x: Between -2.5 and 0.05 with a period of 2pi / 5 seconds (~1.2566)
- y: Between 0.95 and 6.05 with a period of 2pi / 5 seconds (~1.2566)

It's essentially a warped circle where it's more oval shaped and stretched horizontally.

Once the crosshair position has been determined, the normalised direction from the attacker to it determines the angle of the dart's throw. The exact direction is determined by the following 2 vectors:

- From: (attacker's x + 1.0, attacker's y + 2.5)
- To: (crosshair's x, crosshair's y - 0.25)

Effectively, this is the range of the crosshair direction before normalisation:

- x: Between 0.0 and 2.0
- y: Between -1.8 and 3.3

The dart appears at the attacker's `sprite` + (1.0, 0.5, -0.1). The attacker should be at (-3.5, 0.0, -0.65) by then. With that, a target position of the dart is calculated by doing the following:

dart's position + crosshair direction * 15.0 + (0.0, -2.0, 0.0)

The final z angle of the dart is the y of this heading direction - 3.0 and then multiplied by 5.0.

Effectively the z angle is calculated by doing the following:

- The dart's initial y position is 0.5
- It gets added to the normalised crosshair direction * 15.0
- It gets subtracted by -2.0
- It gets subtracted by 3.0
- It gets multiplied by 5.0

This gives an angle between -77.5 and 72.5 in z.

As for the dart's target:

- Initial dart position: (-2.5, 0.5, -0.75) 
- It gets added to the crosshair normalized direction * 15.0
- It gets added to (0.0, -2.0, 0.0)

This gives us the following range:

- x: -2.5 to 8.0
- y: -16.5 to 13.5
- z: -0.75

This is a broad range, but that's because the selection is almost half a circle on the right. What's important is the dart starts near the attacker, goes towards the crosshair (at least close enough) and -2.0 in y.

The dart will move towards its target over the course of 41.0 frames. Every 2 frames (starting from the second one), every enemy party members is checked to see if they can receive damage. The first enemy party member found that can receive damage is the only one that will receive any meaning this is a single target action.

For the enemy party member to receive damage, they need to fufill the following conditions (checked in `enemydata` order):

- Their `hp` must be above 0
- Their [position](../../Actors%20states/BattlePosition.md) must not be `Underground`
- The xy distance between the dart and the enemy party member's position + their `cursoroffset` + (0.25, -1.0 + their `height`, 0.0) must be less than 1.5. NOTE: This is almost the same then the world [CenterPos](../../Actors%20states/CenterPos.md) of the enemy party member, but it is shifted by 0.25 in x so it's slightly off to the right. However, this shift is 1/6th of the radius meaning it barely matters.

As for the damage logic itself, it depends on which hits it is.

For the first one, DoDamage 1 call happens
For the second one, DoDamage 2 call happens
For the third one (if applicable), DoDamage 3 happens

Regardless of which DoDamage call is done:

- PincerStatus is called (see the section below for details)
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called
- The dart movement loop is stopped and the next throw input loop (if applicable) starts

## PincerStatus
Every DoDamage call has [Pierce](../../Damage%20pipeline/AttackProperty.md) meaning they can't inflict any [conditions](../../Actors%20states/Conditions.md) by themselves. However, after each DoDamage call (if applicable), a method called PincerStatus happens which is exclusive to this action and [NeedlePincer](NeedlePincer.md). It implies custom condition infliction logic.

What's unique about this infliction logic is multiple conditions could be inflicted at once and each requires at least having a [medal](../../../Enums%20and%20IDs/Medal.md) equipped for the infliction attempt to occur. Here are the possible conditions and their requirements (not mutually exclusive, checked and attempted in the order presented):

- If `PoisonNeedle` medal is equipped, 2 turns of [Poison](../../Actors%20states/BattleCondition/Poison.md)
- If `ElecNeedles` medal is equipped, 1 turn of [Numb](../../Actors%20states/BattleCondition/Numb.md)
- If `NumbNeedle` medal is equipped and the enemy party member doesn't have the [Numb](../../Actors%20states/BattleCondition/Numb.md) condition, 2 turns of [Sleep](../../Actors%20states/BattleCondition/Sleep.md)

However, the way the infliction attempt is done isn't via [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md), but via a method only used by PincerStatus called [TryCondition](../../Actors%20states/Conditions%20methods/TryConditions.md) which behaves similarily to [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md).

> NOTE: Although similar, [TryCondition](../../Actors%20states/Conditions%20methods/TryConditions.md) has many differences depending on the condition compared to [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) and it even has an off by one error in the resistance check. Check its documentation to learn more

## Logic sequence

- attacker animstate set to 115
- Yield for 0.65 seconds
- attacker's `overridefly` set to true
- attacker animstate set to 117
- The camera moves a bit to the left and back of the stage
- Over the course of 21.0 frames, the attacker moves to be in front of the player party, but still on the left side of the stage while their `height` increases towards 2.75
- All enemy party members whose [position](../../Actors%20states/BattlePosition.md) isn't `Underground` or `OutOfReach` has [TempCrosshair](../../Visual%20rendering/TempCrosshair.md) called on them and their resulting transforms stored in a local array
- Loop performed 2 times (3 times if `Beemerang2` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped):
    - attacker animstate set to 118
    - attacker's `bobspeed` set to 0.1 and `bobrange` to 3.0
    - A local array of 3 SpriteRenderer is initialised. They form the line leading to the crosshair for the targetting selection (read the section above for more details). All of them are from a new ui object named `crosshair` on layer 15 (`3DUI`) rooted:
        - 0: `guisprites[41]` (crosshair without ring)
        - 1: `guisprites[41]` (white hexagon) with a local scale of 0.1875 uniform
        - 2: `guisprites[41]` (white hexagon) with a local scale of 0.375 uniform
    - `Crosshair` sound plays on `sounds[9]` with 0.9 pitch and 0.35 volume on loop
    - The input loop mentioned in the targetting section above occurs and will continue until the Confirm input is pressed in addition that every enemy party members whose `hp` is 0 or below has their TemporaryCursor disabled if they weren't already. Also, the first 2 SpriteRenderer (the hexagons) all rotates by -6.0 in z every frames
    - `sounds[9]` stops looping
    - The normalised direction from `crosshair[0]` (y - 0.25 and z at 0.0) to the attacker (x + 1.0, y + 2.5 and z at 0.0) is obtained
    - All the 3 SpriteRenderer created earlier are destroyed
    - attacker's `overrideanim` is set to true
    - attacker's `bobrange` and `bobspeed` are reset to 0.0
    - attacker animstate set to 119
    - `Toss` sound plays
    - Yield for 0.05 seconds
    - A `projectilepsrites[2]` (needle dart) is created rooted at the attacker position + (1.0, 0.5, -0.1) with a ShadowLite SetUp on it, a scale of 1.15 uniform and z angle determined by the direction of the crosshair (see the targetting selection section for more details)
    - Over a maximum of 41.0 frames, the dart moves towards its target. Every 2 frames (starting from the second one), all enemy party members are checked to see if they will receive damage. See the section above for more details
    - The dart is destroyed
    - Yield for 0.5 seconds if an enemy party member was hit
- All TempCrosshair are destroyed
- Over the course of 16.0 frames, the attacker's `height` decreases to 0.0 and then set to 0.0 after
- attacker's `overrideanim` set to false
- attacker's `overridfly` set to false
