# Unmoveable
Prevents an [Enemy](../NPCType.md#enemy) with this behavior (either default or `inrange`) to not move during the [Dizzy](../Notable%20methods/Dizzy.md) process. This doesn't have any logic, its presence is what affects the NPCControl.

## Frequency meaning
None.

## DoBehavior
No logic occurs here.

## Explanation of the behavior and how it affects [Dizzy](../Notable%20methods/Dizzy.md)
This behavior has no logic, but its presence (either the default or the `inrange` one) affects the [Dizzy](../Notable%20methods/Dizzy.md) method.

Normally, between the 2 yield return null (each being a frame yield), and after the second one, there's several logic there to essentially launch and move the dazed [Enemy](../NPCType.md#enemy). This behavior prevents all this logic to occur with the exception of the HitPart particles which will still play. Besides this, no logic occurs between the 2 frame yields and none after the second one.

The overall effect is while it allows the [Enemy](../Enemy.md) to be dizzy, it will not allow any movement when it happens.