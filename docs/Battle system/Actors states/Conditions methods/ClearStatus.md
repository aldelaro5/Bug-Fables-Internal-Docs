# ClearStatus
This method will clear almost all [conditions](../Conditions.md) off an actor. The actor's [BattleData](../BattleData.md) is received as a ref to allow to reset the entire `condition` list.

Before resetting the `conditions` list, `isnumb`, `isasleep` and [isdefending](../Enemy%20features.md#isdefending) are all set to false.

The method then resets the `condition` list by removing everything except any of the following:

- [EventStop](../BattleCondition/EventStop.md)
- [Eaten](../BattleCondition/Eaten.md)
- [Flipped](../BattleCondition/Flipped.md)
- [Taunted](../BattleCondition/Taunted.md)
- Any condition whose turn count is above 99 (which only happens if the game manually placed it with the intent of being practically infinite), but this doesn't apply to [Poison](../BattleCondition/Poison.md) (meaning the condition inflicted as a result of the `EternalPoison` [medal](../../../Enums%20and%20IDs/Medal.md) is removed despite this exception)

After the reset:

- battleentity.`shieldenabled` is set to false
- `charge` is set to 0
- UpdateConditionIcons is called which calls UpdateConditionBubbles on all battleentity (all `playerdata` with right to false and all `enemydata` with `hp` above 0 with right to true)
