# EndPlayerTurn
This methods tells the `currentturn` player party member that its action is over and its turn must be consumed.

- `playerdata[currentturn].cantmove` is incremented which consumes one of its available action
- `currentturn` is set to -1 which deselects the player party member and lets [Update](Update.md) decide what to do next as part of the [Main turn life cycle](Main%20turn%20life%20cycle.md#main-turn-life-cycle).
- UpdateConditionIcons is called which calls UpdateConditionBubbles on the battleentity (all `playerdata` with right to false and all `enemydata` with `hp` above 0 with right to true)
- [currentaction](../Player%20UI/Pick.md) is set to `BaseAction` (the main vine action menu)
- RefreshAllData is called which sets `alldata` to a new list which consists of all the `playerdata` followed by all the `enemydata` appended together and it also resets all enemy party member's `blockTimes` to 0
- Each `enemydata`'s `blockTimes` gets reset to 0
- Each `enemydata` that has a [Freeze](../Actors%20states/BattleCondition/Freeze.md) condition or `isasleep` while [actimmobile](../Actors%20states/Enemy%20features.md#actimmobile) is false has their `cantmove` set to 1
- SetLastTurns is called which resets `lastturns` to a new aray with the length being the amount of free players - 1 and all elements being -1 (this resets the player selection cycle)
