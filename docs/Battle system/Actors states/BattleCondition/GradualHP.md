# GradualHP
A condition that increases the actor's `hp` by 2 (clamped from 0 to `maxhp`) every actor turn while active.

## [SetCondition](../Conditions%20methods/SetCondition.md)
When amending the condition, the turn amount is always increased by the existing one (meaning it stacks).

When inflicted as a new condition, nothing special happens.

## [AdvanceTurnEntity](../../Battle%20flow/AdvanceTurnEntity.md)
When the condition is processed (when `hp` is above 0, but less than `maxhp`):

- [ShowDamageCounter](../Visual%20rendering/ShowDamageCounter.md) is called with type 1 (HP) with the amount being 2 with a start of the actor position + its `cursoroffset` and an end of Vector3.up
- The actor's `hp` is incremented by 2 clamped from 0 to the actor's `maxhp`
- The `Heal` sound is played
- A yield of 0.75 seconds is set to happen after the method is done
- The turn counter of the condition is decremented
