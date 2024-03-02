# Shield
A condition that encases the acotr in a shield by controlling its battleentity.`shieldenabled` which allows it to bypass damage calculation and not loose its `plating`.

## [Relay](../../Battle%20flow/Action%20coroutines/Relay.md)
This condition will not be transfered to the target of the relay if the relayer had a `RelayTransfer` [medal](../../../Enums%20and%20IDs/Medal.md).

## [SetCondition](../Conditions%20methods/SetCondition.md)
When inflicting or amending the condition:

- battleentity.`shieldenabled` gets set to true
- If the actor is a player party member, the turn counter is overriden to 1

## [DoDamage](../../Damage%20pipeline/DoDamage.md)
This condition allows a target to be considered Invulnerable and bypass damage calculation completely. In such case, the damage ammount is overriden to 0 unless property is `NoExceptions` where it's overriden by the original damageammount sent to the method.

This condition will prevent target.`plating` from being set to false after damage calculation.

This condition prevents the `DefDownOnBlock` and `AtkDownOnBlock` from taking effect after damage calculation which effectively prevents their ensuing [DefenseDown](DefenseDown.md) and [AttackDown](AttackDown.md).

## [AdvanceTurnEntity](../../Battle%20flow/AdvanceTurnEntity.md)
When the condition has just expired, battleentity.`shieldenabled` gets set to false.
