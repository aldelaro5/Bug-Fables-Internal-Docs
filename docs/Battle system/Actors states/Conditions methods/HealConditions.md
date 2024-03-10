# HealConditions
A method that receives an actor's [BattleData](../BattleData.md) as ref and does the following:

- Calls [RemoveCondition](RemoveCondition.md) with the [Freeze](../BattleCondition/Freeze.md) condition on the actor
- Calls [RemoveCondition](RemoveCondition.md) with the [Poison](../BattleCondition/Poison.md) condition on the actor
- Calls [RemoveCondition](RemoveCondition.md) with the [Numb](../BattleCondition/Numb.md) condition on the actor
- Calls [RemoveCondition](RemoveCondition.md) with the [Numb](../BattleCondition/Numb.md) condition on the actor (it is done a second time)
- Set the actor's `isasleep` to false
- Set the actor's `isnumb` to false
