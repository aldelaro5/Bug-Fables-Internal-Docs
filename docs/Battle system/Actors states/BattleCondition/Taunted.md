# Taunted
A condition that prevents a player party member from selecting the `Skill`, `Item` and `Relay` vine menu [Actions](../../Player%20UI/Actions.md) and prevents other player party member from relaying to them.

## [GetChoiceInput](../../Player%20UI/GetChoiceInput.md) 
This condition prevents the following [Actions](../../Player%20UI/Actions.md) option (the buzzer is played instead):

- `Skill`
- `Item`
- `Relay` 

This condition will also deny the confirmation of another player party member's choice when selecting who to relay to when handling the `Relay` choice for a `SelectPlayer` action. Effectively, it means not only the condition prevents the player party member from relaying, it also prevents them to be relayed to.

## [SetMaxOptions](../../Player%20UI/SetMaxOptions.md)
This condition causes the vine icons for these [Actions](../../Player%20UI/Actions.md) to render with the MainManager.`grayscale` material instead of MainManager.`spritedefaultunity`. 

## [ClearStatus](../Conditions%20methods/ClearStatus.md)
This condition is excluded from removal meaning it will remain even after calling this method.

## [UpdateAnim](../../Visual%20rendering/UpdateAnim.md)
For all player party members  with an `hp` above 4 with this condition and whose battleentity.`overrideanim` is false, their battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) may be set to AngryAnim which depends on the [animid](../../../Enums%20and%20IDs/AnimIDs.md): 10 (`Flustered`) for `Bee`, 5 (`Angry`) for `Beetle` and 102 for `Moth`. Check the method's documentation for the exact conditions.

TODO: no enemy logic?