# Reflection
A condition that causes a player party member to sustain less damage from [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) with the reduction amount being the amount of `Reflection` [medal](../../../Enums%20and%20IDs/Medal.md) equipped. It always expire on the next main turn, but the turn counter is used to visually show the amount of the damage reduction.

## [Relay](../../Battle%20flow/Action%20coroutines/Relay.md)
This condition will not be transfered to the target of the relay if the relayer had a `RelayTransfer` [medal](../../../Enums%20and%20IDs/Medal.md).

## Choosing [DoNothing](../../Player%20UI/ItemList%20confirmation%20handling/Battle%20strategy%20list%20type.md#2-do-nothing) in the [Battle strategy list type](../../../ItemList/List%20Types%20Group%20Details/Battle%20Strategy%20List%20Type.md)
If the `currentturn` player party member has a `Reflection` [medal](../../../Enums%20and%20IDs/Medal.md), this condition is inflicted via [SetCondition](../Conditions%20methods/SetCondition.md) with the turn amount being the amount of `Reflection` equipped (this isn't a true turn counter and more a visual to show the amount of damage reduction that will happen when attacked).

## [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md)
This condition will cause a decrease of the damage for a player party member. The reduction amount is the amount of `Reflection` [medal](../../../Enums%20and%20IDs/Medal.md) equipped on the target.

## [AdvanceTurnEntity](../../Battle%20flow/AdvanceTurnEntity.md)
When the condition is processed (when `hp` is above 0), the turn counter is always set to 0. This is because this condition is meant to expire immediately since the actual turn counter is semantically used to show the amount of damages any attacks will be reduced by.
