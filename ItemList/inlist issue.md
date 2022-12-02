# inlist issue

The ItemList system has a complex issue with `inlist` as it is not always properly cleared after an ItemList is processed which can lead to issues when using the [Pickitem](../SetText/Commands/Individual%20commands/Pickitem.md) command from [SetText](../SetText/SetText.md). This issue has as of 1.1.2 2 known reproduction under normal gameplay, but there used to have 3 known reproduction as the first known one was patched in 1.1.0. It is however affecting [SetText](../SetText/SetText.md) and [ItemList](ItemList.md) as a whole and its complexity is worth to be thoroughly explained.

## The intent of inlist

`inlist` is used alongside [ItemList](ItemList.md) to determine if an ItemList is currently being processed, but `inlist` is a boolean intended to be true as long as the ItemList is being processed while [ItemList](ItemList.md) is the game object of the current ItemList which is intended to tell if a list is active. This is an important distinction because a [Pickitem](../SetText/Commands/Individual%20commands/Pickitem.md) command relies on `inlist` staying true after [ItemList](ItemList.md) goes to null to know that an ItemList has been confirmed or cancelled. When this happens, the ItemList gets destroyed, but the handling of the choice has yet to the handled by SetText.

In other words, `inlist` should remain true as long as an ItemList is being processed and go to false once it has been processed even if the actual ItemList might no longer be active.

## The issue with inlist

The problem lies with `inlist` not having a guarantee to be set to false once processing is completed. While most situations such as any [Pickitem](../SetText/Commands/Individual%20commands/Pickitem.md) command offers this guarantee, the same cannot be said for direct calls to [ShowItemList](ShowItemList.md) and the situations that do offer such guarantees are done because a workaround was applied by having the caller set it to false. The only guarantee offered by `inlist` is that it will be set to false once done at the end of a [Pickitem](../SetText/Commands/Individual%20commands/Pickitem.md) command.

This creates an issue where the following occurs in order:

* [ShowItemList](ShowItemList.md) is called directly which will set `inlist` to true
* The ItemList is done processing, but `inlist` remains true
* Any SetText calls in [Dialogue mode](../SetText/Dialogue%20mode.md) occurs whose input string starts with anything except [Pickitem](../SetText/Commands/Individual%20commands/Pickitem.md)
* The first char loop iteration gets done and now we are in [life cycle > Dialogue post-processing](../SetText/life%20cycle.md#dialogue-post-processing) while [ItemList](ItemList.md) is null and `inlist` is true which is treated as if an ItemList is ready to be handled while no such ItemList happened during the course of this SetText call
* If `listredirect` was set from a previous [Pickitem](../SetText/Commands/Individual%20commands/Pickitem.md) command, SetText will redirect to it which leads to undefined behaviors depending on what value it had before

## Mitigations in place

While this is a global issue affecting the interactions between both systems, there are some mitigations put in place that workarounds the issue in most situations.

The first one is setting `inlist` to false at specific points such as when pausing or doing any actions in battle that isn't an item usage. There is also a workaround applied in the case an abomination is used leading to a game over where `listredirect` is set to -1 before the SetText call is done. This workaround was applied in 1.1.0.

## Known reproductions

Despite all those measures, there are 2 known cases to reproduce this issue:

1. Have all turns in battle be item usage and use Chompy's change ribbon's function. This one is special as it is possible to reproduce the issue completely inside a battle because Chompy's change ribbons uses a [Pickitem](../SetText/Commands/Individual%20commands/Pickitem.md) with the [Chompy Ribbons List Type](List%20Types%20Group%20Details/Chompy%20Ribbons%20List%20Type.md).
1. Have all turns in battle be item usage until an EventDialogue is triggered with a SetText call in [Dialogue mode](../SetText/Dialogue%20mode.md). If the trigger requires to damage the enemy, all damages must be from items uses.

## Possible fixes

While it is possible to apply workarounds as reproductions are found by the caller or after the ItemList is fully handled, it is possible to permanently fix this issue.

The first way would be to guarantee that `inlist` is set to false during the [dialogue setup phase](../SetText/Life%20Cycle/dialogue%20setup%20phase.md) of SetText. This can work because it is not possible to make use of the value until a [Pickitem](../SetText/Commands/Individual%20commands/Pickitem.md) is processed in which case, it is dangerous to leave it dangling at a false value. While it would still leave the value dangling in the end, any potential problems will be negated immediately by the next SetText call in [Dialogue mode](../SetText/Dialogue%20mode.md).

The second way would be to clear `listredirect` right after a redirection is performed if any. This wouldn't entirely fix the problem, but it will neuter the potential problems to a safer level.
