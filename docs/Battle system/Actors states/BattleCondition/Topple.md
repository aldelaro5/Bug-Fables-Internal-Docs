# Topple
A 1 hit defensive mechanism in the form of a condition for an enemy party member where if it has a `ToppleFirst` or `ToppleAirOnly` (if their `position` is `Flying`) [weakness](../../Damage%20pipeline/AttackProperty.md) it won't get dropped or [Flipped](Flipped.md) immediately when it was supposed to otherwise. Instead, this condition will be inflicted without the drop or flip featuring a `Woobly` [animstate](../../../Entities/EntityControl/Animations/animstate.md) and it will be removed after most attacks which allows the drop or flip to occur on any subsequent hit.

## [Relay](../../Battle%20flow/Action%20coroutines/Relay.md)
This condition will not be transfered to the target of the relay if the relayer had a `RelayTransfer` [medal](../../../Enums%20and%20IDs/Medal.md). It should be noted that it is not possible to inflict this condition on a player party member under normal gameplay so this clause effectively does nothing.

## [DoAction](../../Battle%20flow/Action%20coroutines/DoAction.md)
Before an enemy party member acts, if it has this condition or [Flipped](Flipped.md) for exactly 1 actor turn left, both are removed alongside the following logic:

- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) is called on the enemy party member with a height of 10.0
- entity.`overrideanim` is set to false
- entity.`basestate` and entity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 0 (`Idle`)
- startstate is set to 0 (`Idle`)
- 0.1 seconds are yielded

## [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md)
This condition prevents toppling an enemy party members with a `ToppleFirst` or `ToppleAirOnly` [AttackProperty](../../Damage%20pipeline/AttackProperty.md) in their `weakness` (because it would be redundant).

This condition is removed if [Freeze](Freeze.md) is inflicted.

This condition is removed if the target has it, property is `Flip` while the target has a `Flip` and `ToppleFirst` [weakness](../../Damage%20pipeline/AttackProperty.md) without the [Flipped](Flipped.md) condition. It essentially acts as replacing `Topple` with `Flipped` in this case since it also inflicts [Flipped](Flipped.md). However, if the condition isn't present while the other conditions are fufilled, it is inflicted for 1 turn and on top of this, if the the target.`position` is `Ground`, battleentity.`basestate` is set to 21 (`Woobly`). 

This condition is removed if the damage calculation logic needs to drop the enemy party member. Check the method documentation to learn about the conditions for that to happen.

## [Drop](../../../Entities/EntityControl/Notable%20methods/Drop.md)
For any enemy party member entity with this condition after the drop, it is removed alongside setting its `basestate` to 0 (`Idle`).

## [DoDamageAnim](../../Visual%20rendering/DoDamageAnim.md)
This condition may cause the actor's battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) and `basestate` to be set to 21 (`Woobly`) alongside setting `overrideanim` to false. Check the method's documentation to learn more.
