# EventStop
A stop condition specifically made for [event](../../../Enums%20and%20IDs/Events.md) 182 (trying to exit the room after approaching the tank in the last room of Upper Snakemouth) that simply disables a party member until it is manually removed (it is not meant to expire naturally).

## [IsStopped](../IsStopped.md)
This condition is considered a stop condition and will always make this method returns true (unless skipimmobile is false while the actor'a `actimmobile` is true). This include the lite version used in the [enemy phase](../../Battle%20flow/Main%20turn%20life%20cycle.md#enemies-phase). Being stopped makes the actor unable to act regardless of their `cantmove` as well as a bunch of feature they no longer get access to.

## [GetFreePlayerAmmount](../Player%20party%20members/GetFreePlayerAmmount.md)
This condition makes a player party member not count as free. This affects many logic such as knowing if a player can act.

## [GetChoiceInput](../../Player%20UI/GetChoiceInput.md) (`Relay` -> `SelectPlayer`)
This condition will deny the confirmation of a player party member when selecting who to relay to when handling the `Relay` choice for a `SelectPlayer` action.

## [Relay](../../Battle%20flow/Action%20coroutines/Relay.md)
This condition will not be transfered to the target of the relay if the relayer had a `RelayTransfer` [medal](../../../Enums%20and%20IDs/Medal.md). It should be noted that it is not possible to act with this condition on a player party member under normal gameplay so this clause does nothing in practice.

## [AdvanceTurnEntity](../../Battle%20flow/AdvanceTurnEntity.md)
When the condition just expired after decrementing its turn counter:

- [BreakIce](../../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md) is called on the actor's battleentity if its `icecube` exists
- The actor's `cantmove` is set to 1 (this will become 0 if their `hp` is above 0 since turn advancement has yet to happen)

If the condition didn't expire after the decrement, `cantmove` is set to 1 for an enemy party member or to 0 for a player party member instead of the normal actor turn advancement.

NOTE: This logic should normally never apply because the condition is made to not expire and be manually removed.

## [ClearStatus](../Conditions%20methods/ClearStatus.md)
This condition is excluded from removal meaning it will remain even after calling this method.

## [UpdateAnim](../../Visual%20rendering/UpdateAnim.md)
For all alive (`hp` above 0) player party members with this condition and whose battleentity.`overrideanim` is false, their battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 11 (`Hurt`).
