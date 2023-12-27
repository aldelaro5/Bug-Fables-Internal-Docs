# RemoveCondition
This method removes a [condition](../Conditions.md) and also perform the associated side effects involved with removing it. It belongs to MainManager.

```cs
public static void RemoveCondition(BattleCondition condition, BattleData entity)

```

## Parameters

- `condition`: The condition to remove if it exists
- `entity`: The actor to remove the condition from

## Procedure

The condition is searched and if it's not found, nothing happens.

If it's found, some specific `BattleCondition` has a side effect for its removal that will be performed:

- `Numb` or `Sleep`: RefreshCondition is called on the actor's battleentity which updates the actor's `isasleep` and `isnumb` fields according to an [HasCondition](HasCondition.md) call for `Sleep` and `Numb` respectively.
- `Freeze`: The actor's entity (not its battleentity) has [BreakIce](../../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md) called on it. TODO: for a player party member, this might be useless and have side effects on the overworld entity, recheck

NOTE: If multiple occurences of the condition exists, only the first one is removed.
