# Skills list type confirmation handling
This page describes the logic [SetItem](../SetItem.md) has when handling a [listtype](../../../ItemList/listtype.md) of [Skills](../../../ItemList/List%20Types%20Group%20Details/Skills%20List%20Type.md).

- `maxoptions` is set to the length of `availabletargets` (NOTE: this should be up to date as [SetTargets](../../Actors%20states/Targetting/SetTargets.md) was called the last time [PlayerTurn](../../Battle%20flow/PlayerTurn.md) happened while we were in the main vine menu)
- `tempskill` is set to the sent `listvar` option which is the chosen [skill](../../../Enums%20and%20IDs/Skills.md) id
- `currentchoice` is set to `Skill`
- `helpboxid` is set to the one that comes from the [skilldata](../../../TextAsset%20Data/Skills%20data.md#skilldata), but if MainManager.`mashcommandalt` is true (sequential keys is set in the settings):
    - 4 (`TappingKey`) is overriden to 8 (`RandomTappingKeysTimer`)
    - 10 (`RandomTappingBar`) is overriden to 8 (`RandomTappingKeysTimer`)
    - 11 (`RandomPressKeyTimer`) is overriden to 8 (`RandomTappingKeysTimer`)
    - 9 (`RandomPressBar`) is overriden to 12 (`MultiPressBar`)
- `itemarea` is set to the `AttackArea` from `skilldata`
- `excludeself` is set to the the value from `skilldata`
- If `itemarea` is `User`, there is no need to have [GetChoiceInput](../GetChoiceInput.md) handle this so it is handled immediately by starting a [DoAction](../../Battle%20flow/Action%20coroutines/DoAction.md) action coroutine changing to an [uncontrolled flow](../../Battle%20flow/Update%20flows/Uncontrolled%20flow.md)
- Otherwise, [GotoSelect](../GotoSelect.md) is called with overridemax
