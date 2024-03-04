# `SelectEnemy` ChoiceInput logic
This page details the logic of [GetChoiceInput](../GetChoiceInput.md) when `currentaction` is `SelectEnemy`.

[CreateHelpBox](../../Visual%20rendering/CreateHelpBox.md) is called followed by `excludeself` being set to false.

The rest is input handling logic.

## Input 2 / 3 (left / right)
These 2 inputs are only processed if `itemarea` isn't `AllParty`, `AllEnemies` or `All`.

These inputs changes the `option` by one depending on the direction (decrement if left, increment if right with wrap around from 0 to `maxoptions` - 1 using DecreaseOption and IncreaseOption). 

Additionally, a `Scroll` sound played on `sounds[10]` before changing the `option` and [UpdateText](../../Visual%20rendering/UpdateText.md) is called after changing it.

## Input 5 (cancel)
ReturnToMainSelect is called which does the following:

- The `Cancel` sound is played on AudioSource 10
- `currentaction` is set to `BaseAction`
- `option` is set to `lastoption`
- `selecteditem` is set to -1
- DestroyHelpBox is called which sets `helpboxid` to -1 and destroys `helpbox` if it existed in 0.5 seconds with shrink before setting it to null
- [SetMaxOptions](../SetMaxOptions.md) is called
- [UpdateText](../../Visual%20rendering/UpdateText.md) is called

## Input 4 (confirm)
A `Confirm` sound is played on `sounds[10]` followed by `target` being set to `option`.

What happens after depends on the `currentchoice` (nothing happen if it's not among these `Actions`).

### `Attack`
A [DoAction](../../Battle%20flow/Action%20coroutines/DoAction.md) action coroutine is started changing to an [uncontrolled flow](../../Battle%20flow/Update.md#uncontrolled-flow) on `playerdata[currentturn].battleentity` using -1 as the action id.

### `Item`
CheckItemUse is called with the `selecteditem` which ends starting a [UseItem](../../Battle%20flow/Action%20coroutines/UseItem.md) action coroutine changing to an [uncontrolled flow](../../Battle%20flow/Update.md#uncontrolled-flow).

### `Skill`

- `lastskill` is set to `selecteditem`
- A [DoAction](../../Battle%20flow/Action%20coroutines/DoAction.md) action coroutine is started changing to an [uncontrolled flow](../../Battle%20flow/Update.md#uncontrolled-flow) on `playerdata[currentturn].battleentity` using `selecteditem` as the action id.

### `Strategy`

- [ItemList](../../../ItemList/ItemList.md)'s `listredirect` is set to -1 (this workarounds a potential [inlist issue](../../../ItemList/inlist%20issue.md))
- `currentaction` is set to `BaseAction`
- A [Tattle](../../Battle%20flow/Action%20coroutines/Tattle.md) action coroutine is started changing to an [uncontrolled flow](../../Battle%20flow/Update.md#uncontrolled-flow) if `disablespy` is false.
