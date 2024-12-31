# CleanKill
This is a method that turns an enemy party member into a blank shell meaning they are thoroughly put out of commision, disappear and can no longer have effects on the battle, but may still remain in `enemydata` if `selfsacrifice` or `inevent` is true. It is meant to be used for enemy actions or [EventDialogue](../Battle%20flow/EventDialogue.md).

```cs
private void CleanKill(ref MainManager.BattleData target, Vector3 startp)
```

## Parameters

- `target`: The actor to turn into a blank shell. NOTE: While it is possible to send a player party member, it is untested and doesn't seem to be useful for this
- `startp`: The position to place the battleentity after killing them

## High level explanation
This method solves an edgecase where [CheckDead](../Battle%20flow/Action%20coroutines/CheckDead.md)'s normal death routines would be undesired to kill an enemy party member during their action or during an [EventDialogue](../Battle%20flow/EventDialogue.md). 

Normally, CheckDead looks at the enemy party member's [deathtype](Enemy%20features.md#deathtype) to determine how to kill the enemy alongside the [Death](../../Entities/EntityControl/Notable%20methods/Death.md) process at the entity level which looks at the `destroytype` (which is mapped from the `deathtype`). The problem is this enforces only one way to kill an enemy party member: it can be desired to simply have the enemy disappear instantly and to not have them be in the battle anymore.

While it could be possible to kill them manually by calling Death on their battleentity, this gets complicated because this wouldn't actually remove the `enemydata` as this is done by a separate method: [ReorganizeEnemies](Enemy%20party%20members/ReorganizeEnemies.md). CheckDead normally runs it so it could be possible to call Death on them and THEN run a CheckDead (who will skip the enemy party member because their `dead` field is now true) or a ReorganizeEnemies, but now a bigger problem emerges during [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md): what if the enemy party member need to kill themselves in this fashion DURING their action?

This is a very unsafe thing to do because there are several places in [post action](../Battle%20flow/Action%20coroutines/DoAction.md#post-action) where the game assumes the enemy still exists. If a ReorganizeEnemies is ran during the action, the post action phase is now susceptible to incorrectly index `enemydata` because it would try to access an enemy party member that no longer exists! This can in the best case mess with the wrong enemy party member which can make the logic inconsistent or in the worst case, an exception being thrown leading to a stall since DoAction hasn't had the chance to set `action` to false leading the battle stuck perpetualy in an [uncontrolled flow](../Battle%20flow/Update%20flows/Uncontrolled%20flow.md).

This is where this method comes in: it will cleanly put an enemy party member out of commision, but allow the caller to not run a ReorganizeEnemies if they set `selfsacrifice` to true. Doing this also has an effect on the post action because it also prevents some logic to run which wouldn't make sense if the enemy decided to kill themselves. The overall effect is the `enemydata` elements is thoroughly killed, not even CheckDead will detect it because it was already killed. Crucially however, the `enemydata` element STILL EXISTS which allows the post action to not run into problems (and the logic remains consistent because it is reporting accurate information as the enemy just died very thoroughly by this method).

However, ReorganizeEnemies still needs to eventually run because leaving the blank shell in causes a whole host of problem. Fortunately, this isn't an issue: CheckDead is guaranteed to run as one of the last step of the post action phase and it is late enough that the game can guarantee the safety since it no longer needs to access the enemy party member. CheckDead in this case will simply ignore the enemy as their `dead` field is true (so it doesn't kill them again) and it will also call ReorganizeEnemies as a normal part of its procedure which completely removes the blank shell.

There is an exception to this: [EventDialogue](../Battle%20flow/EventDialogue.md). ReorganizeEnemies won't run during those either due to `inevent` being true. This is acceptable because EventDialogue needs much more control over how they can kill enemy party members and they can anyway decide to call the method when needed. This is also why CheckDead also won't unconditionally call it when `inevent` is true: the EventDialogue can decide what happens to an enemy that just died as a result of their [eventondeath](Enemy%20features.md#eventondeath) triggering.

NOTE: Under normal gameplay, ReorganizeEnemies is never called by this method. This is because it is always called during an EventDialogue or during an enemy action to kill themselves after setting `selfsacrifice` to true.

## Lack of Destroy
While this method will result in the enemy party member getting removed from `enemydata` by [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md)'s post action, the same can't be said for its `battleentity` from the scene: it WILL NEVER get destroyed as a result of CleanKill.

This is because it is due to a side effect of the `None` [destroytype](../../Entities/EntityControl/Notable%20methods/Death.md#none) since it bypasses the call to Destroy the `battleentity`. Due to the safety concerns mentioned above, this is actually correct for CleanKill itself since it's likely unsafe to do this destruction at this stage. In summary, this method CANNOT guarantee the destruction of the `battleentity` and it is the responsibility of the caller to ensure it gets destroyed later when it becomes safe to do so (typically, after ReorganizeEnemies ran, but this implies a reference to the `battleentity` was obtained before).

In practice however, none of the CleanKill callers ever destroys it manually even though it should be their responsibility to do it once it becomes safe to do so. In other words, due to an oversight with this method's limitation, every CleanKill call under normal gameplay will cause the enemy party member's `battleentity` to remain in the scene even after it is removed from `enemydata`.

## Procedure
The following happens on the target enemy party member:

- If `shadow` exists, it is disabled
- All `condition` are manually cleared
- [ClearStatus](Conditions%20methods/ClearStatus.md) is called on the enemy party member
- [UpdateAnim](../Visual%20rendering/UpdateAnim.md) is called
- DestroyConditionIcons is called on the enemy party member
- All `statusicons` are destroyed then the array is set to null
- [deathtype](Enemy%20features.md#deathtype) set to 0
- `destroytype` set to [None](../../Entities/EntityControl/Notable%20methods/Death.md#none)
- `sprite` gets disabled
- `startscale` set to Vector3.zero
- `exp` set to 0
- `hp` set to 0
- `money` set to 0
- position set to startp
- `nocondition` set to true (prevents to render status icons)
- [LockRigid](../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) is called to lock the `rigid` in place
- `height` set to 999.0
- StartDeath called which sets `deathcoroutine` to a [Death](../../Entities/EntityControl/Notable%20methods/Death.md) call on the enemy party member

Finally, if `selfsacrifice` and `inevent` are both false (which effectively doesn't happen under normal gameplay), [ReorganizeEnemies](Enemy%20party%20members/ReorganizeEnemies.md) is called with order.
