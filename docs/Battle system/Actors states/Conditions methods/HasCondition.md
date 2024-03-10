# HasCondition
This methods checks the presence of a [condition](../Conditions.md) and returns the amount of actor turns left on it if it exist or -1 if it doesn't. It belongs to MainManager. This is the main way to check the presence of any conditions in the battle system.

```cs
public static int HasCondition(BattleCondition condition, BattleData entity)
```

### Parameters

- `condition`: The `BattleCondition` to check the presence of
- `entity`: The actor to check the condition for

### Procedure
The method goes over all `condition` elements of entity and if one is found with the condition sent, true is returned, false otherwise.

There is a side effect to this method: if the actor has the [Sleep](../BattleCondition/Sleep.md) condition, it may set its `isasleep` to true. Here is all the circumstances that will have that happen:

- [Sleep](../BattleCondition/Sleep.md) was sent in as the condition
- We aren't looking for `Sleep`, but the condition we are looking for occurs after a `Sleep` one
- We are looking for a condition that doesn't exist

This means that it is possible to have `isasleep` not be set to true if we are looking for a condition that occurs before a [Sleep](../BattleCondition/Sleep.md) one. In that case, the method returns true without leaving a chance for `isasleep` to be set to true. In practice however, this doesn't cause any problems.
