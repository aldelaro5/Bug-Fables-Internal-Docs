# AlwaysWander
A very cut down version of [wander](Wander.md) where the entity will always, when it can, move somewhere on the next DoBehavior cycle even if it is in the process of moving already.

> NOTE: this is unused under normal gameplay and is likely not fully functional despite having unique logic backing this behavior.

## Frequency meaning
The same as [wander](Wander.md), but it's not useful in practice because this behavior ignores the `actioncooldown`.

## SetUp (Only if it's the default behavior)
There is no logic here unlike [wander](Wander.md) because this behavior ignores `actioncooldown`.

## DoBehavior
It starts off the same than [wander](Wander.md) until when it comes time to decide the main logic branch to take.

This behavior will always take the main one and it will always be seen as if the `actioncooldown` expired. There is a twist however: when entering the branch, there is now an early exit if the player is `tattling` or we are in a `pause` where [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity and this DoBehavior cycle ends early.

The rest is the same as the main logic of wander. 

## Explanations of the changes
This has the implications that the NPCControl will always wander if all of the conditions are met. These conditions depends if the entity is in a `forcemove` or not.

If they aren't in a `movemove`, the player needs to not be `tattling` while we aren't in a `pause` to wander on this cycle.

If they are in a `forcemove`, the above applies on top of all of the following needing to be true:

- entity.`hitwall` is false
- entity.`onground` is true
- entity.`detect` is absent or [HasGroundAhead](../../EntityControl/EntityControl%20Methods.md#hasgroundahead) from the entity.`forcetarget` returns true

If the conditions aren't met in either case, the DoBehavior cycle ends early with a [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) call on the entity.

Because the entity will always try to wander and it will in most practical cases, it essentially makes the NPCControl move nonstop, even during its movement. It means it's very possible for it to try to move to different points every frame which is rather erratic. It is likely why this behavior was left unused as it doesn't seem to be fully functional in this state and its usage isn't recommended.