# SetItem
This method is a special handler whenever a confirmation of an [ItemList](../../ItemList/ItemList.md) occurs. The [listtype](../../ItemList/listtype.md) handled here are only the ones related to BattleControl except for the [chompy ribbons list type](../../ItemList/List%20Types%20Group%20Details/Chompy%20Ribbons%20List%20Type.md) as this one is handled by the [Chompy](../Battle%20flow/Action%20coroutines/Chompy.md) action coroutine. The ItemList handled are only the ones that were setup by [GetChoiceInput](GetChoiceInput.md)

The method receives the `listvar` option that was confirmed which is set to `selecteditem`. `turncooldown` is set to 5.0 and `option` to 0. The `turncooldown` prevents [GetChoiceInput](GetChoiceInput.md) to be called from [PlayerTurn](../Battle%20flow/PlayerTurn.md) for 5 [FixedUpdate](../Visual%20rendering/FixedUpdate.md) cycles.

From there, this is where the list type is handled. Only 3 are possible:

- [Battle strategy list type](../../ItemList/List%20Types%20Group%20Details/Battle%20Strategy%20List%20Type.md)
- [Skills list type](../../ItemList/List%20Types%20Group%20Details/Skills%20List%20Type.md)
- [Standard items list type](../../ItemList/List%20Types%20Group%20Details/Items%20List%20Type.md)

The logics are located in their own page matching the corresponding `listtype`. No matter which one, [UpdateText](../Visual%20rendering/UpdateText.md) is called after.

|listtype|Link to its UI handling logic page|
|-------:|---------------------------------|
|Battle strategy|[Battle strategy list type](ItemList%20confirmation%20handling/Battle%20strategy%20list%20type.md)|
|Skills|[Skills list type](ItemList%20confirmation%20handling/Skills%20list%20type.md)|
|Standard items|[Standard items list type](ItemList%20confirmation%20handling/Standard%20items%20list%20type.md)|