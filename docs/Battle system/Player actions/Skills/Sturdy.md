# `Sturdy`

This action features no action commands or damages logic. It only does the following:

- The target player party member is selected to be the attacker if it's a skill usage or the `target` if it's an ItemUsage (see the section below for details). It is possible that the `target` is the attacker if the item was used on themselves
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) is called on the target to inflict the [Sturdy](../../Actors%20states/BattleCondition/Sturdy.md) conditiion for 1 turn
- If the absolute value of the attacker's `cantmove` is above 0, [Heal](../../Actors%20states/Heal.md) is called on the target to heal it by that amount. NOTE: This is incorrect. The heal was supposed to be for the absolute value of `cantmove` when it is negative because this would have healed by the amount of extra actor turns, but this logic will incorrectly heal extra inactive actor turns (for example, a `cantmove` of 1 would heal by 1 despite the target being unable to act already)
- The target's `lockcantmove` is set to true
- The target's `cantmove` is set to 1. NOTE: This is incorrect if the attacker is the target because the actor turn of the attacker will be consumed in [post-action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action) making its `cantmove` go to 2 while the main turn will advance it to 1 which would have made them unable to act. The task below does workaround this, but in a disruptive and unecessary way
- If this is a skill usage or it's an ItemUsage where the attakcer is the target, the attacker's `moreturnnextturn` is incremented. NOTE: This is incorrect, but it workarounds the issue mentioned above with `cantmove`. The correct logic was to set `cantmove` to 0 if the attacker is the `target`, but the game rather set it to 1 and advances the leftover inactive actor turn via `moreturnnextturn` which allows the actor to act again on the next main turn

## [Battle](../../Battle%20flow/Action%20coroutines/UseItem.md#battle) ItemUsage
This action will be processed if a `ShellOil` [item](../../../Enums%20and%20IDs/Items.md) is used with a `Battle` ItemUsage or any [item](../../../Enums%20and%20IDs/Items.md) with a `Sturdy` ItemUsage (the latter is never used under normal gameplay). This manner of triggering the action is the only one that allows to have a different target than the attacker which has different logic implications as mentioned above. It is assumed for the skill usage that the attacker is the target while the ItemUsage may or may not be.

## Issue with `dontusecharge`
This action doesn't deal damages to an enemy party member, but it doesn't set `dontusecharge` to true. This means this action will incorrectly consume the `charges` accumulated on the attacker. This includes the ItemUsage mentioned above which means that using a `SheilOil` also incorrectly consume `charges` on the user.

## startstate changes
This action changes the startstate to 24 (`Block`).

## Logic sequence

- The target player party member is selected to be the attacker if it's a skill usage or the `target` if it's an ItemUsage (see the section below for details). It is possible that the `target` is the attacker if the item was used on themselves
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) is called on the target to inflict the [Sturdy](../../Actors%20states/BattleCondition/Sturdy.md) conditiion for 1 turn
- Target `overrideanim` set to true
- Target animstate set to 4 (`ItemGet`)
- Target y `spin` set to 30.0
- Yield for 0.75 seconds
- Target `spin` zeroed out
- `StatUp` sound plays
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on target with type 1 (blue up arrow)
- Target animstate and startstate set to 24 (`Block`)
- Yield for 0.5 seconds
- The target's `lockcantmove`, `cantmove` and `moreturnnextturn` are modified as explained above
