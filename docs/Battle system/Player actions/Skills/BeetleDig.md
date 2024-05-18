# `BeetleDig`

## [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|60.0|[PressKeyTimer](../../Action%20commands/PressKeyTimer.md)|{Random between Confirm, Cancel and Switch inputs id}|

## [DamageWithinPos](../../Damage%20pipeline/DamageWithinPos.md) calls

|#|Conditions|playerid|damageammount|property|radius|xonly|
|-:|---|---|---|---|---|---|
|1|Done for each party member who ended up fufilling the [IsInRadius](../../Damage%20pipeline/IsInRadius.md) check after target selection (see the section below for details)|`currentturn`|`playerdata[currentturn].atk` clamped from 1 to 99. NOTE: This clamp is unecessary because of the property which already clamps the final value from 1 and it also incorrectly removes the influences of any base `atk` value of 0 or below|[Atleast1pierce](../../Damage%20pipeline/AttackProperty.md)|2.0|true|
|2|Done for each enemy party member who fufills these conditions:<ul><li>DamageWithinPos 1 applied</li><li>DoCommand 1 set `commandsuccess` to true</li></ul>|`currentturn`|`playerdata[currentturn].atk` / 1.5 rounded to nearest clamped from 1 to 99. NOTE: This clamp is unecessary because of the property which already clamps the final value from 1 and it also incorrectly removes the influences of any base `atk` value of 0 or below|[Atleast1pierce](../../Damage%20pipeline/AttackProperty.md)|2.0|true|

## About the target selection
There is a "fake" action command in this action that happens before DamageWithinPos 1, but its outcome influences both DamageWithinPos calls because it involves the attacker's position moving horizontally back and forth until the Confirm input is pressed. Since the method will use the player party member's position as the point to compare with the radius, this is effectively a target selection.

It works by having an indefinite loop that won't stop yielding frames until the Confirm input is pressed. Before each yield, the attacker's y/z positions are zeroed out, but the x is lerped from the existing one to sin curve being 3.75 + Sin(Time.time * 2.0) * 7.0 with a factor of 0.025.

> NOTE: Mathematically, this lerp makes no sense. Using simulation reveals the curve changes phase shift and amplitude and it can approximated by 3.75 + sin(2 * Time.tim - 0.92) * 4.23 but this is framerate dependant. Had the curve been followed correctly, it would have actually covered too large of a range, but the lerp accidentally fixes it despite making no sense mathematically.

Once the input is pressed the x position is determined, it won't change and it will be used for both DamageWithinPos calls, but with xonly to true so only the x distance is checked to fall within 2.0.

## Additional flip logic
Neither of the DamageWithinPos calls comes with a `Flip` [property](../../Damage%20pipeline/AttackProperty.md), but the action comes with custom logic that does a reduced version of what the property did. It only concerns the [conditions](../../Actors%20states/Conditions.md) management and the `basestate` set in case of a [Topple](../../Actors%20states/BattleCondition/Topple.md).

The logic is contained in a method called FlipWithinPos which is only used in this action. The method takes the x position of the attacker after it was determined using the logic mentioned in the section above and find every enemy party members who satisfies all the following conditions:

- Their [position](../../Actors%20states/BattlePosition.md) isn't `OutOfReach`
- Their `hp` is above 0
- They are [IsInRadius](../../Damage%20pipeline/IsInRadius.md) (without getair and xonly) of (x position of the attacker, 0.0, 0.0) where the radius is 2.0. NOTE: This should have had xonly as true, but it ends up being a non issue in practice because getair being false means that the physical position of `Flying` enemy party members will count as being in radius if close enough. Since all `Flying` enemy party members uses their `height` field to render in the air and their physical position remains at 0 in y and z, this ends up not affecting the logic badly

For each enemy party members found, Flip is called on them. This method features the same [flip handling](../../Damage%20pipeline/CalculateBaseDamage.md#flip-handing) logic than the damage pipeline, but with 3 key differences:

- It doesn't have any logic about the damage effects (it doesn't apply here)
- It doesn't feature a [DropItem](../../Actors%20states/Enemy%20party%20members/DropItem.md) call if the enemy party member was holding an item
- It doesn't feature setting `isdefending` to false meaning it will not break their guarding

The rest (including the toppling management and the [Flipped](../../Actors%20states/BattleCondition/Flipped.md) condition infliction) are the same.

## Logic sequence

- attacker's `shieldenabled` value is saved and then set to false which visually hides the shield (it will be restored when the action is over and it doesn't affect the logic, only visuals)
- `Dig` sound played
- `checkingdead` set to a KabbuDig coroutine which initiates a digging animation of the attacker with particles, `Dig` sound, sprite local position/angles changes and set `overrideanim`, `overrideflip` and `overrideheight` to true. The coroutine lasts 40.0 frames and sets `checkingdead` to null at the end
- Yield until `checkingdead` goes to null
- attacker animstate set to 1 (`Walk`)
- `Digging` sound plays on `sounds[9]` at 0.75 volume on loop
- `Prefabs/Particles/Digging` instantiated childed to the attacker
- attacker ForceMove to (-0.5, 0.0, 0.0) over the course of 40.0 frames with both movestate and stopstate to 1 (`Walk`)
- Yield for a frame
- Yield until attacker's `forcemoving` goes to null
- A new Confirm ButtonSprite is SetUp without description childed to the GUICamera at (-6.0, 2.0, 10.0) with a sortorder of 10
- The camera slowly moves to look at the right side of the stage
- Yield for a frame
- All enemy party members whose [position](../../Actors%20states/BattlePosition.md) isn't `Underground` or `OutOfReach` has [TempCrosshair](../../Visual%20rendering/TempCrosshair.md) called on them and their resulting transforms stored in a local array
- The target selection input logic mentioned above takes place, but with the added logic that the ButtonSprite's `basesprite`.color periodically switches between pure white and pure grey where it's the former when Sin(Time.time * 5.0) * 10.0 is higher than 0.0, it's the latter otherwise. This won't proceed until the Confirm input is pressed.
- All TempCrosshair are destroyed
- `sounds[9]` is stopped
- `DigPop` sound plays
- attacker animstate set to 119
- ButtonSprite's `basesprite`.color is set to pure white then disabled
- ShakeScreen with 0.25 ammount for 0.55 time
- attacker's `overrideheight` set to false
- attacker's `overridejump` set to true
- `Prefabs/Particles/Digging` positioned offscreen
- `DireExplode` particles plays
- DoDamageWithinPos 1 call happens
- The FlipWithinPos logic mentioned above happens
- attacker gets LockRigid(true) called
- attacker's `overrideflip` set to false
- attacker's y `spin` set to 20.0
- Over the course of 61.0 frames, the attacker moves along a BeizierCurve where both ends are its current position, but with a ymax of 10.0 meaning they will move up to 10.0 and down back to where they started. After half of the frames passed, the following happens on the first applicable frame once:
    - DoCommand 1 call happens
    - Yield for a frame
    - Yield until `doingaction` goes to false
    - If `commandsuccess` is true:
        - `Turn` sound plays
        - instance.`camoffset` increases by (0.0, 0.5, -0.5)
        - attacker animstate set to 101
        - attacker's `spin` zeroed out
        - Over the course of 21.0 frames, the attacker moves along a BeizierCurve where it goes from its current position to the same position + (0.0, 1.0, 0.0)
        - attacker animstate set to 119
        - attacker's `sprite`.flipY set to true
        - attacker's y `spin` set to 30.0
        - `Woosh4` sound played
        - Over the course of 16.0 frames, attacker moves to 0.0 in y and z
        - attacker's y position set to -2.0 and z position set to 0.0
        - `Dig` sound plays
        - `DirtExplode` particles plays at attacker's x position (0.0 in y and z) for 1.0 seconds
        - ShakeScreen with 0.35 ammount for 0.85 time
        - DamageWithinPos 2 call happens
        - Yield for 1.0 seconds
        - break out of the ascension loop early
    - attacker's `sprite`.flipY set to true
    - attacker's `height` set to 1.0
- attacker's `overrideanim` set to false
- attacker's `overridejump` set to false
- `Prefabs/Particles/Digging` repositioned locally at Vector3.zero
- attacker position reset to what it was before the ascencion
- attacker's `overrideheight` set to true
- attacker's `sprite` y local position set to -2.0 and flipY to false
- attacker gets LockRigid(false) called
- Camera reset to default
- attacker MoveTowards its position before this action
- Yield while attacker is done with its `forcemove`
- `DirtExplode` particles plays with `DigPop` sound at the attacker's position
- attacker's `overrideheight` set to false
- attacker's `height` reset to 0.0
- `Prefabs/Particles/Digging` destroyed
- ButtonSprite destroyed
- attacker's `shieldenabled` reset to the value at the start of this action
