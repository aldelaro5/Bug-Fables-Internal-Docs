# GetChoiceInput
This method manages the UI naviguation called from [PlayerTurn](../Battle%20flow/PlayerTurn.md) when `turncooldown` expired.

The logic depends on the `currentaction` and most of the logic involves input handling through the different menus. The logics are located in their own page matching the corresponding [Pick](Pick.md) value handled by this method (the others are handled somewhere else such as MainManager in the case of an [ItemList](../../ItemList/ItemList.md)).

|`currentaction`|Link to its UI handling logic page|
|--------------:|---------------------------------|
|`BaseAction`|[BaseAction](Confirmation%20handling/BaseAction.md)|
|`SelectEnemy`|[BaseAction](Confirmation%20handling/SelectEnemy.md)|
|`SelectPlayer`|[BaseAction](Confirmation%20handling/SelectPlayer.md)|
