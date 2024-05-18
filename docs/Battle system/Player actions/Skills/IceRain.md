# `IceRain`

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Done 4 times for each enemy party members that fufills these conditions:<ul><li>Their `hp` is above 0</li><li>Their [position](../../Actors%20states/BattlePosition.md) isn't `OutOfReach` or `Underground`</li><li>They are [IsInRadius](../../Damage%20pipeline/IsInRadius.md) (without getair and xonly) with a radius of 2.0 of the crosshair's x position (see the section below for more details)</li></ul>|`playerdata[currentturn]`|The enemy party member|`playerdata[currentturn].atk` unclamped|[Freeze](../../Damage%20pipeline/AttackProperty.md)|Empty array|false|

## About the target selection
There is a "fake" action command in this action that happens before all the DoDamage 1 calls, but its outcome influences all of these calls because it involves the position used for [IsInRadius](../../Damage%20pipeline/IsInRadius.md) moving horizontally back and forth until the Confirm input is pressed. Since the method will change the position of the radius, this is effectively a target selection.

To indicate the x position, a new UI object is created called `crosshair` rooted on layer 15 (`3DUI`) with a SpriteRenderer using the sprite `guisprite[41]` (a crosshair without a ring).

Before each DoDamage call, there will be an indefinite loop that won't stop yielding frames until the Confirm input is pressed. Before each yield, the `crosshair`'s z angle increases by 10x the game's frametime, y position is set to 6.0 and the z to 0.0, but the x is the result of a sin curve. That curve is 3.5 + Sin(Time.time * 5.0) * 3.0 which means the `crosshair`'s x position will cycle from 0.5 to 6.5 with a period of 2pi / 5 seconds (~1.2566 seconds).

Once the input is pressed the `crosshair` x position is determined and the z is fixed to 0.0, but the y has some special logic to it. It will try to go towards 0.0, but it's possible the it won't be 0.0. It happens if the x of the `crosshair` was less than 1.0 away from the x world [CenterPos](../../Actors%20states/CenterPos.md) x of any enemy party member checked in `enemydata` order. In that case, the y component will be the first y position of an icicle object dropping from 6.0 to 0.0 where the value becomes less than the enemy party member's y world CenterPos. This drops occurs in a span of 46.0 frames. Intuitively, it can be thought of that if an object would fall on top of an enemy party member, the `crosshair` will stop short instead of going to the ground at 0.0. This can affect the radius check.

Once the `crosshair` position is fully determined, it won't change and it will be used for the next DoDamage call and apply to each enemy party members that fall within 2.0. of this point. NOTE: The [IsInRadius](../../Damage%20pipeline/IsInRadius.md) call has getair to false meaning it will accept either the physical position or the world [CenterPos](../../Actors%20states/CenterPos.md) of the enemy party member.

This process is repeated for each instances of DoDamage 1 calls so each uses a different base point to check for the radius that is controlled by the player.

## Logic sequence

- Camera moves towards the right side of the stage
- attacker animstate set to 105
- Yield for 0.5 seconds
- The crosshair mentioned in the section above is created
- A local array of 4 GameObject is initialised which will contain the 4 icicles used for the action
- All enemy party members whose [position](../../Actors%20states/BattlePosition.md) isn't `Underground` or `OutOfReach` has [TempCrosshair](../../Visual%20rendering/TempCrosshair.md) called on them and their resulting transforms stored in a local array
- Loop done exactly 4 times:
    - `Crosshair` sound played on `sounds[9]` with 0.9 pitch and 0.35 volume on loop
    - attacker animstate set to 105
    - Yield for a frame
    - The target selection loop mentioned in the section above occurs. During it, every enemy party members whose `hp` is 0 or below will have its TempCrosshair disabled if it wasn't already
    - The main `crosshair` used for the target selection is disabled
    - `sounds[9]` stops playing
    - The corresponding element of the icicle array is initialised to a new `Prefabs/Objects/icecle` rooted at (main `crosshair`'x position, 15.0, 0.0)
    - An IceRain corotine starts with a startpos of the `crosshair`'s x position and the newly created `Prefabs/Objects/icecle` as icicle (see the section below for details)
    - attacker animstate set to 100
    - Yield for 0.5 seconds
- All TempCrosshair are destroyed
- attacker animstate set to 102
- Yield until icicles used are completely null (they are supposed to be destroyed, but it can take some time before they get set to null)
- Yield for 0.5 seconds

### IceRain
This is a coroutine that is specific to this action. It is started after each Confirm input press and it is the coroutine that performs the icicle drop mentioned earlier as well as the DoDamage 1 call.

```cs
private IEnumerator IceRain(float startpos, Transform icecle)
```

## Parameters

- `startpos`: The `crosshair`'s x position used for targetting like mentioned earlier
- `icicle`: An icicle object to drop, has to be a different instance on each call (which it is normally)

## About possible race conditions
This coroutine is never yielded on: it simply starts with DoDaction not paying close attention on it. The only thing DoAction does after starting it is to yield for 0.5 seconds, but this coroutine is designed to take longer than that.

However, this is safe to do this: the icicle parameter are different elements of an array and each calls gets a different one. The coroutine limits itself to perform tasks that doesn't affect what DoAction is doing so even if they are both running concurently, in practice, this is safe to do so. The only effect it has is it allows for the player to drop the next icicle slightly before the previous one's drop ended. This is again, safe because the coroutine doesn't introduce any race conditions with DoAction.

Additionally, the last step of the coroutine involves destroying the icicle and DoAction makes sure to yield until all of the ones that were passed to this coroutine are set to null meaning all instances of the coroutines are guaranteed to be done after these yields on DoAction's end.

## Procedure

- Each enemy party members is checked if they are so close to the icicle horizontally that the drop should be stopping short if it touches that enemy party member. See the section above about target selection to learn more, but the result is stored in the form of the `enemydata` index if it applies or -1 if it doesn't
- Over the course of 46.0 frames, the icile drop occurs where the icicle moves towards 0.0 in y with its y angle increased by 20x the game's frametime each frames. This is also where the target position can stop short in y if the icicle falls below the concerned enemy party member (see the section about target selection above to learn more)
- `mothicenoraml` particles played at the target position + (0.0, 1.0, 0.0) with a scale of (2.0, 2.0, 2.0)
- For each targetted enemy party members that satisfied the DoDamage 1 call's conditions, the call happens with them
- The icicle is moved offscreen
- Yield for 0.5 seconds
- The icicle is destroyed (DoAction will wait that all of them are fully set to null)
