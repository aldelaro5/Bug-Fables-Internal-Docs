# LateUpdate
The LateUpdate of BattleControl is very minor. Most of it involves updating some HUD visibility.

The following HUD are updated if we aren't in a [terminal flow](../Battle%20flow/Update%20flows/Terminal%20flow.md):

- instance.`showmoney` (the berry counter HUD) is set to -1.0 (hidden) if `action` is false (meaning we are in a [controlled flow](../Battle%20flow/Update%20flows/Terminal%20flow.md), but this doesn't cover [EventDialogue](../Battle%20flow/EventDialogue.md))
- instance.`hudcooldown` is set to 10.0 if the [message](../../SetText/Notable%20states.md#message) lock is released

Finally, if `superblockedthisframe` is above 0.0, it is decreased by framestep. This update the amount of frames a super block counts as it was originally determined by [GetBlock](../Battle%20flow/GetBlock.md) and first processed by [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md).