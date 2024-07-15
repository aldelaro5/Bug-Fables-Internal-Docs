# HPPercent
A method that returns an actor's `hp` / `maxhp` which is the percentage (between 0.0 and 1.0) of `hp` they have left. It is primarily used in enemy actions as a factor to enemies's logic.

```cs
private float HPPercent(MainManager.BattleData data)
```

## Parameters

- `data`: The actor to obtain their `hp` remaining percentage

## Procedure
Returns data's `hp` / `maxhp`. It can then be used as a percentage of `hp` left so for example, a return of 0.5 means 50% of `hp` is left, 0.0 means their `hp` is 0 and 1.0 means their `hp` is the same than their `maxhp`.
