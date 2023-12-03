# DisguiseOnce
The same than [Disguise](Disguise.md), but if it's the default behavior, it becomes [Wander](Wander.md) whenever the player gets `inrange` and the `disguiseobj` is destroyed.

## Frequency meaning
None.

## Update (active, only if it is the default behavior)
If we aren't `trapped`, we are `inrange`, and none of the override cases applied if this is an [Enemy](../NPCType.md#enemy) (dizzy, frozen and about to be unfrozen), the default behavior is set to [Wander](Wander.md) and the `disguiseobj` is destroyed if it was present which ends this behavior.