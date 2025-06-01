# HasWeakness
This is a method that belongs to MainManager:

```cs
public static bool HasWeakness(BattleControl.AttackProperty property, BattleData entity)
```
Returns true if `entity`.`weakness` contains `property`, false otherwise.

An enemy party member's `weakness` list is the list of [AttackProperty](../../Damage%20pipeline/AttackProperty.md) that applies on the enemy party member which changes how they behave, particularily within [DoDamage](../../Damage%20pipeline/DoDamage.md).
