# IsStopped
This is a method that determines if an actor is unable to act.

```cs
private bool IsStopped(MainManager.BattleData entity, bool skipimmobile)
```

## Parameters

- `entity`: The actor to check
- `skipimmobile`: Whether or not to exclude the `actimmobile` check. There is an overload without this parameter that sends false

## Procedure

If skipimmobile is false and the actor's `actimmobile` is true, the result is false.

Otherwise, the result is only true if any of the following are true (it's false otherwise):

- The actor has a `Freeze` [condition](Conditions.md)
- The actor has a `Numb` [condition](Conditions.md)
- The actor has a `Flipped` [condition](Conditions.md)
- The actor has a `Sleep` [condition](Conditions.md)
- The actor has a `Eaten` [condition](Conditions.md)
- The actor has a `EventStop` [condition](Conditions.md)
- The actor `isnumb`
- The actor `isasleep`
- The actor's `hp` is 0 or below
