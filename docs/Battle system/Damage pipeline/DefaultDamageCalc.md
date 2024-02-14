# DefaultDamageCalc
This is a sub method to [CalculateBaseDamage](CalculateBaseDamage.md) which only gets called for most property values (the exceptions being `NoExceptions` where it's never called and `Flip` where it may or may not be called).

It takes the basevalue as ref and despite its name, it only processes optional decreases of it due to defensive effects.

```cs
private void DefaultDamageCalc(MainManager.BattleData target, ref int basevalue, bool pierce, bool blocked, int def)
```

## Parameters:

- `target`: The target from CalculateBaseDamage
- ref `basevalue`: The basevalue from CalculateBaseDamage
- pierce: Whether defense pierce applies. This value is computed from CalculateBaseDamage, but overriden to false if `target` is a player party member
- `blocked`: UNUSED (this is block from CalculateBaseDamage, but it is never read in the method)
- `def`: target.`def` from CalculateBaseDamage (NOTE: this is the base value, not the effective one). Overriden to 0 if `pierce` is true while `target` is an enemy party member

## Procedure

- pierce is overriden to false if the target is a player party member
- If pierce is still true, def is overriden to 0
- If the target doesn't have the `Flipped` [condition](../Actors%20states/Conditions.md), basevalue is decreased by def
- If target is a player party member:
    - If the target has the `Poison` [condition](../Actors%20states/Conditions.md), basevalue is decreased by (the amount of `PoisonDefender` [medals](../../Enums%20and%20IDs/Medal.md) equipped on the target - the amount of `ReversePoison` [medals](../../Enums%20and%20IDs/Medal.md) equipped on the target)
    - If the target is positioned last in the player party, basevalue is decreased by the amount of `BackSupport` [medals](../../Enums%20and%20IDs/Medal.md) equipped on the target
    - If the target is positioned in front of the player party, basevalue is decreased by the amount of `FrontSupport` [medals](../../Enums%20and%20IDs/Medal.md) equipped on the target
    - If the target.`hp` is 4 or less, basevalue is decreased by 2 * the amount of `DefenseUp` [medals](../../Enums%20and%20IDs/Medal.md) equipped on the target
    - If the target has a `HeavySleeper` [medal](../../Enums%20and%20IDs/Medal.md) equipped while having a `Sleep` [condition](../Actors%20states/Conditions.md), basevalue is divided by 2 (integer division so it's floored implicitly)
- If target.`isdefending` is true while pierce is false, basevalue is decreased by target.`defenseonhit`
