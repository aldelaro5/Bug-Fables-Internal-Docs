# HealConditions
A method that receives an actor's BattleData as ref and does the following:

- Calls [RemoveCondition](RemoveCondition.md) with the `Freeze` condition on the actor
- Calls [RemoveCondition](RemoveCondition.md) with the `Poison` condition on the actor
- Calls [RemoveCondition](RemoveCondition.md) with the `Numb` condition on the actor
- Calls [RemoveCondition](RemoveCondition.md) with the `Numb` condition on the actor
- Set the actor's `isasleep` to false
- Set the actor's `isnumb` to false