# Drop
This coroutine handles dropping an airborne enemy in the air during battle. It only applies when the entity isn't `dead`, `iskill`, `deathcoroutine` isn't running and it has a valid `battleid`. If not all these conditions are met, `droproutine` is set to null before a yield break. Note that `droproutine` must be stored externally by the caller.

## Drop preparation
First, `overrideanim` is temporarily set to false (the old value is saved). Then, the [animstate](../Animations/animstate.md) is constantly set to Hurt with a frame yield until the current battle's startdrop goes to true.

From there, there is an early exit path if the enemy's hp reached 0 while not in a battle event. If this happen, `droproutine` is set to null followed by a yield break.

After, `temphgeightoverride` is set to true if the entity was frozen. This is going to come into play later because being frozen implicates different levels of ground.

There is a special case that follows if the `originalid` is the `VenusGuardian` [AnimID](../../../Enums%20and%20IDs/AnimIDs.md) where both of her extra (her leaf wings) will have the `leafexplode` particle play at their position before being disabled. This will also reset the battle camera to default.

For all cases, the `Fall` sound plays if it wasn't playing already. Then, the `rigid` velocity is zeroed out followed by `overrideanim` being set to true (this is still temporary).

There is a special case for the `Spuder` [AnimID](../../../Enums%20and%20IDs/AnimIDs.md) where his `line` is destroyed if it was present followed by his `basestate` going back to Idle.

## The actual drop
`bobrange` and `bobspeed` are both zeroed out to avoid any bobbing in the air as the entity needs to look frozen in place. From there, unless `nofallfrozen` is true (which bypass the actual drop), the `height` is smoothly decreased such that the `spritetransform`'s y position goes below `minheight` unless `tempheightoverride` is true which changes the target y to 0.0. This is because when an entity is frozen, the cube cannot be offset from the ground so it must go to 0.0 and not `minheight`. [BreakIce](Freeze%20handling.md#breakice) will undo this override when appropriate. Additionally, the rate of descent increases over time. It starts at framestep * 0.1, but each frame, this goes up by 10% making the `height` decrease faster each frames by 10%. This keeps going until the y position reached a low enough value (this will be changed periodically due to [UpdateHeight](../Update%20process/UpdateHeight.md)).

After, if `shakeondrop` is true, 4 things happens in a row:

* The particle `impactsmoke` is played at the transform
* The screen is shook by (0.5, 0.5, 0.5) over 0.45 seconds
* The sound `Thud3` is played
* 0.25 seconds are yielded

Finally, the battle position of the entity with its corresponding `battleid` is set to Ground before yielding 0.4 seconds.

## Post drop cleanup
`bobspeed` and `bobrange` are restored to their start field counterpart if the final `height` is still above 0.1. Then, `overrideanim` is restored to its original value before the drop.

If the battle data at `battleid` has the Topple condition, it is removed and the `basestate` is reset to Idle. Them, the [animstate](../Animations/animstate.md) is reset to `basestate` except for the `CursedSkull` [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) which has it set to 101.

Finally, `droproutine` is set to null and the Drop call is over.
