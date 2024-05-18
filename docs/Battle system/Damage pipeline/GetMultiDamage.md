# GetMultiDamage
A method that allows to precalculate the sum of all damages that could happen from [CalculateBaseDamage](CalculateBaseDamage.md) had multiple player party members combined their attacks and damage influence. The method simulates damage effects as if it was done with a player attacker for each player party members involved and adds them together before returning the result. 

The intent with this method is to use the precalculated number in [DoDamage](DoDamage.md) as the damageammount, but without an attacker. This allows to have every effects from the target's side apply, but to not reapply the effects the attacker side would have because DoDamage no longer has an attacker. It acts as a stopgap to have multiple player party members be the "attackers" which isn't possible with DoDamage alone. This method has some caveats, see the details below.

```cs
private int GetMultiDamage(int[] ids)
```

## Parameters

- `ids`: The [trueid](../playerdata%20addressing.md#by-the-player-party-members-trueid) arrays of the player party members that will be involved in the multi damage simulation. Due to this only working for the player party, only 0, 1 or 2 are valid elements (`Bee`, `Beetle` and `Moth` respectively)

## Procedure

The following is a table of all the effects applied to the result in order for each player party members whose `trueid` is in ids and the conditions for the effect to apply. The result is a local value starting at 0 and the effect column describes what happens relative to the player party member involved in the calculation. The sum of this table for all involved player party members is what will be returned as the result clamped from 1 to 99. 

> NOTE: This clamp at the end can skew the results because it negates the influence of having all involved player party members with 0 or negative `atk` value with no bonuses. However, it should be noted that unlike most player actions, this precalculates the EFFECTIVE damage instead of the base one that is typically sent to [DoDamage](DoDamage.md) in a player to enemy flow. Due to this, the bad influence of this clamp is close enough to what would have happened if DoDamage was called with `AtLeast1` or `AtLeast1Pierce` as the [property](AttackProperty.md) which is well defined.

|Conditions|Result effect|
|----------|-------------|
|None|+ `atk`|
|None|+ `charge`|
|The player party member has the [AttackUp](../Actors%20states/BattleCondition/AttackUp.md) condition|+ 1|
|The player party member has the [AttackDown](../Actors%20states/BattleCondition/AttackDown.md) condition|- 1|
|The player party member has the [Poison](../Actors%20states/BattleCondition/Poison.md) condition|+ (amount of `PoisonAttacker` [medals](../../Enums%20and%20IDs/Medal.md) equipped - amount of `ReversePoison` [medals](../../Enums%20and%20IDs/Medal.md) equipped) NOTE: The `ReversePoison` influence here is incorrect, see the caveats section below for details|
|The player party member's `hp` is 4 or less|+ amount of `AttackUp` [medals](../../Enums%20and%20IDs/Medal.md) equipped|
|None|- `tired`|

### Known caveats
There are a few known problems with this precalculation scheme when calling DoDamage without an attacker where the logic differs than the regular damage pipeline:

- The `ReversePoison` [medals](../../Enums%20and%20IDs/Medal.md) should normally have removed 1 defense, but here, it penalises the raw damages done from the player party side which is the same then removing 1 attack which isn't what the medal was supposed to do
- This method doesn't account for the front bonus CalculateBaseDamage normally accounts for and it cannot be accounted for because DoDamage has to be called without an attacker while a player attacker is required to process the front bonus
- This method doesn't manage the logic related to the `LifeSteal` [medal](../../Enums%20and%20IDs/Medal.md) and it cannot be accounted for because DoDamage has to be called without an attacker while a player attacker is required to process the effects of this medal
