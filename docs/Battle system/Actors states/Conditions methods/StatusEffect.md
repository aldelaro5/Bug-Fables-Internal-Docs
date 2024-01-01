# StatusEffect
This method is kind of a wrapper around [SetCondition](SetCondition.md) because it does end up calling it after a resistance check if applicable, but it also renders visually the [conditions](../Conditions.md) being applied by a colored arrow thanks to [StatEffect](../../Visual%20rendering/StatEffect.md).

```cs
private void StatusEffect(MainManager.BattleData data, MainManager.BattleCondition status, int turns, bool effect)
```

### Parameters

- `data`: The actor to attempt to inflict the condition
- `status`: The condition to attempt to inflict the actor
- `turns`: The amount of actor turns to inflict the condition
- `effect`: Whether the infliction should include a visual render. There is an overload where this can be ommited and it will send false

### Procedure

If applicable, a resistence check will be performed. The test will be performed by generating a random integer in the range \[0, 100\[ and check if the number is strictly higher than the matching resistance. However, if the resistance is 100 or above, the test always fails.

> NOTE: this test is off by one numerically: a 50 resistance actually means a 49% change to inflict for example. This is because doing a strictly higher than comparison splits the ranges like such that the fail group is \[0, resistance\] and the success group is \]resistance, 100\[, but they should be split to \[0, resistance\[ and \[resistance, 100\[ respectively. This means for a given resistance, the chance to inflict in percentage is 100 - resistance - 1 instead of 100 - resistance.

Here are the statuses where the test is applicable and which resistance will be used for each:

- `Poison`: `poisonres`
- `Sleep`: `sleepres`
- `Numb`: `numbres`
- `Freeze`: `freezeres`

If the test fails, nothing happens and a return happens immediately.

If it succeeds, [SetCondition](SetCondition.md) is called with the actor, condition and amount of actor turns. This does imply that another resistance check will happen if it amends the condition, but it doesn't matter because it will always pass given the previous test.

The method is done if effect is false.

If it's true, a [StatEffect](../../Visual%20rendering/StatEffect.md) coroutine will be started if applicable with a type followed by a certain sound being played if it wasn't already depending on the condition.

Here are the conditions and how the visual effect is done (other conditions don't have visual effects):

- `AttackUp`: Type 0 with the `StatUp` sound
- `AttackDown`: Type 2 with the `StatDown` sound
- `DefenseUp`: Type 1 with the `StatUp` sound
- `DefenseDown`: Type 3 with the `StatDown` sound
