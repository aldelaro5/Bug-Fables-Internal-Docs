# RemoveCondition
This method removes a [condition](../Conditions.md) and also perform the associated side effects involved with removing it. It belongs to MainManager.

NOTE: If multiple occurences of the condition exists, only the first one is removed.

```cs
public static void RemoveCondition(BattleCondition condition, BattleData entity)
```

## Parameters

- `condition`: The condition to remove if it exists
- `entity`: The actor to remove the condition from

## Procedure

The condition is searched and if it's not found, nothing happens.

If it's found, some specific `BattleCondition` has a side effect for its removal that will be performed:

- [Numb](../BattleCondition/Numb.md) or [Sleep](../BattleCondition/Sleep.md): RefreshCondition is called on the actor's battleentity which updates the actor's `isasleep` and `isnumb` fields according to an [HasCondition](HasCondition.md) call for [Numb](../BattleCondition/Numb.md) and [Sleep](../BattleCondition/Sleep.md) respectively.
- [Freeze](../BattleCondition/Freeze.md): The actor's entity (not its battleentity) has [BreakIce](../../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#breakice) called on it. NOTE: This is technically wrong because it calls the method on the overworld entity instead of the battle one, but this will effectively not do anything destructive since the `icecube` of the entity is null. It however means that the caller is instead responsible for calling the method correctly which the game does in every circumstances under normal gameplay.
