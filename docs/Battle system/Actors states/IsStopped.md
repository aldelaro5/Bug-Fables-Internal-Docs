# IsStopped
This is a method that determines if an actor is unable to act.

```cs
private bool IsStopped(MainManager.BattleData entity, bool skipimmobile)
```

## Parameters

- `entity`: The actor to check
- `skipimmobile`: Whether or not to exclude the [actimmobile](Enemy%20features.md#actimmobile) check. There is an overload without this parameter that sends false

## Procedure

If skipimmobile is false and the actor's [actimmobile](Enemy%20features.md#actimmobile) is true, the result is false.

Otherwise, the result is only true if any of the following are true (it's false otherwise):

- The actor has a [Freeze](BattleCondition/Freeze.md) condition
- The actor has a [Numb](BattleCondition/Numb.md) condition (or `isnumb` is true)
- The actor has a [Flipped](BattleCondition/Flipped.md) condition
- The actor has a [Sleep](BattleCondition/Sleep.md) condition (or `isasleep` is true)
- The actor has a [Eaten](BattleCondition/Eaten.md) condition
- The actor has a [EventStop](BattleCondition/EventStop.md) condition
- The actor's `hp` is 0 or below
