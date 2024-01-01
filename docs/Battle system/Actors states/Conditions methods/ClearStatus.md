# ClearStatus
This method will clear almost all conditions off an actor. The actor's `BattleData` is received as a ref to allow to reset the entire `condition` list.

The method first resets the `condition` list by removing everything except any of the following:

- `EventStop` condition
- `Eaten` condition
- `Flipped` condition
- `Taunted` condition
- Any condition whose turn count is above 99 (which only happens if the game manually placed it with the intent of being practically infinite), but this doesn't apply to `Poison` (meaning the condition inflicted as a result of the `EternalPoison` [medal](../../../Enums%20and%20IDs/Medal.md#medal--badge) is removed despite this exception)

Then:

- battleentity.`shieldenabled` is set to false
- `charge` is set to 0
- UpdateConditionIcons is called which calls UpdateConditionBubbles on all battleentity (all `playerdata` with right to false and all `enemydata` with `hp` above 0 with right to true)