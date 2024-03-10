# GetFreePlayerAmmount
This method returns the amount of `playerdata` that are considered free. It belongs to MainManager.

This amount includes all player party members have the following criteria:

- `cantmove` is 0 or below (at least one action is available)
- `hp` is above 0 (the player isn't dead)
- It does not have any of the following [conditions](../Conditions.md):
    - [Sleep](../BattleCondition/Sleep.md)
    - [Numb](../BattleCondition/Numb.md)
    - [Freeze](../BattleCondition/Freeze.md)
    - [EventStop](../BattleCondition/EventStop.md)
    - [Eaten](../BattleCondition/Eaten.md)
