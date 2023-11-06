# DisguiseOnce
The same than [Disguise](Disguise.md), but if it's the default behavior, it becomes [Wander](Wander.md) whenever the player gets `inrange`.

## Frequency meaning
None ???

## Start / SetUp
The same then [Disguise](Disguise.md).

## Update (Active, main, only if it is the default behavior)
If we aren't `trapped`, we are `inrange`, and none of the override cases applied if this is an `Enemy` (dizzy, frozen and about to be unfrozen), it is set to [Wander](Wander.md) and the `disguiseobj` is destroyed if it was present which ends this behavior.

## DoBehavior
The same then [Disguise](Disguise.md).

## Update (Inactive)
The same then [Disguise](Disguise.md).

## LateUpdate (Not a `dummy` and the entity is `incamera`)
The same then [Disguise](Disguise.md).

## Update ([Dizzy](../Notable%20methods/Dizzy.md) `Enemy` override)
The same then [Disguise](Disguise.md).
