# CancelList
This method is used to go back to the main vine action menu from an [ItemList](../../ItemList/ItemList.md) during a battle.

- `option` is set to `lastoption`
- `currentaction` is set to `BaseAction` (the main vine action menu)
- `turncooldown` is set to 5.0 (this prevents [GetChoiceInput](GetChoiceInput.md) calls during [PlayerTurn](../Battle%20flow/PlayerTurn.md) for 5 [FixedUpdate](../Visual%20rendering/FixedUpdate.md) cycles)
- [SetMaxOptions](SetMaxOptions.md) is called
- [UpdateText](../Visual%20rendering/UpdateText.md) is called
