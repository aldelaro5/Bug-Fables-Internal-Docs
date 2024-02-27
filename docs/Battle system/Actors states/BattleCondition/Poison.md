# Poison
A condition that either deals non lethal damage or heals the actor each actor turn depending on if they have a `ReversePoison` [medal](../../../Enums%20and%20IDs/Medal.md) or not. It also allows several poison specific medals to take effect.

## Resistance
This condition has a dedicated resistance field for actors to use: `poisonres`. If it's 100 or above, the actor is immune to it. Inflictions that requires a resistance check will use this field.

## [SetCondition](../Conditions%20methods/SetCondition.md)
When amending the condition: 

- If the actor is a player party member with an `EternalPoison` [medal](../../../Enums%20and%20IDs/Medal.md), the turn amount is overriden to be 99999 and the infliction overwrites the turn counter to the new one (meaning it won't stack)
- Otherwise, the inflicting is ammended by adding the turn amount to the existing one (meaning it will stack)

When inflicted as a new condition, the same `EternalPoison` logic occurs (the turn ammount is overriden to 99999).

## [StatusEffect](../Conditions%20methods/StatusEffect.md)
This condition is supported by this method and it might be inflicted to the actor when called.

## [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md)
This condition may be inflicted if the property is `Poison`. This means it's also supported by the `StatusMirror` [medal](../../../Enums%20and%20IDs/Medal.md).

This condition allows the `PoisonAttacker` [medal](../../Enums%20and%20IDs/Medal.md) to take effect by increasing the damage amount by the amount of `PoisonAttacker` equipped for a player attacker while not in `demomode`. 

A variation of the `PoisonAttacker` logic is also enforced (except the `demomode` check) outside of the damage pipeline in GetMultiDamage with the only difference being that the increase is by (amount of attacker's `PoisonAttacker` - amount of attacker's `ReversePoison`). That method is used by the following [skills](../../../Enums%20and%20IDs/Skills.md):

- `IceSphere`
- `IceDrill`
- `BeeFly`

### [DefaultDamageCalc](../../Damage%20pipeline/CalculateBaseDamage.md#defaultdamagecalc)
This condition allows the `PoisonDefender` and `ReversePoison` [medals](../../Enums%20and%20IDs/Medal.md) to take effect by decreasing the damage amount by (amount of target's `PoisonDefender` - amount of target's `ReversePoison`).

## [DoDamage](../../Damage%20pipeline/DoDamage.md)
This condition allows the The `PoisonTouch` [medal](../../Enums%20and%20IDs/Medal.md) to take effect is equipped on the target if the following are all true:

- The attacker is an enemy party member
- The attacker's `poisonres` is less than 100 (it's not immune)
- `nonphysical` is false

If all of the above are fufilled, [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called to set the `Poison` [condition](../Actors%20states/Conditions.md) on the corresponding enemy party member of the attacker for 2 turns.

## [AdvanceTurnEntity](../../Battle%20flow/AdvanceTurnEntity.md)
When the condition is processed (when `hp` is above 0 and there is at least one enemy party member):

- If the actor is a player party member with a `ReversePoison` [medal](../../Enums%20and%20IDs/Medal.md) equipped:
    - The actor's `hp` is increased by the actor's `maxhp` * 0.1 rounded to nearest clamped from 1 to 3. NOTE: the nearest rounding has a quirk where if it ends in .5, the even number will be chosen over the odd one no matter if it is actually the lower or higher bound that is correct mathematically
    - If the `Heal` sound isn't playing or its more than halfway done player, it is played
    - [ShowDamageCounter](../Visual%20rendering/ShowDamageCounter.md) is called with type 1 with the damage amount as the amount healed earlier with the start being the battleentity position + (0.0, 1.0, 0.0) and the end being (0.0, 2.0, 0.0)
- Otherwise, [DoDamage](../Damage%20pipeline/DoDamage.md) is called with target being the actor with no attacker with a `NoExceptions` property without block. The damageammount is the actor's `maxhp` / 10.0 ceiled - 1 and then clamped from 1 to 3 for a player party member and from 1 to 2 for an enemy party member. The call also has these overrides:
    - `NoFall`,
		- `NoIceBreak`,
		- `FakeAnim`,
		- `DontAwake`,
		- `IgnoreNumb`
- If the actor's `hp` became 0, battleentity.`overrideanim` is set to false followed by an OverrideOver call being invoked on the battleentity in 1.0 seconds
- The actor's `hp` is clamped from 1 to its `maxhp` (this implies it can't be lethal)
- A yield of 0.75 seconds is set to happen after the method is done

## [HealConditions](../Conditions%20methods/HealConditions.md)
This condition is supported by this method and it will be removed when called.

## [UpdateAnim](../../Visual%20rendering/UpdateAnim.md)
This condition will cause the `currentturn` player party member to have its battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) set to 20 (`WeakPickAction`) instead of (`PickAction`) when its battleentity.`overrideanim` is false.

This condition may cause the other player party members to have their battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) set to 17 (`WeakBattleIdle`) instead of 13 (`BattleIdle`). Check the method's documentation to learn more on the specifics.

## [UpdateEntities](../../Visual%20rendering/UpdateEntities.md)
This condition will have the battleentity.`sprite`.material.color to be set to a lerp from the existing one to 0xD800D8 (bright magenta) with a factor of 1/5 of the frametime. Check the documentation of the method to learn more about how the alpha channel is determined.
