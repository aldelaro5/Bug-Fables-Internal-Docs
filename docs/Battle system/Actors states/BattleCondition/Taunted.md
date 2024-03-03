# Taunted
A condition that prevents a player party member from selecting the `Skill`, `Item` and `Relay` vine menu [Actions](../../Player%20UI/Actions.md) and prevents other player party member from relaying to them.

## [GetChoiceInput](../../Player%20UI/GetChoiceInput.md) 
This condition prevents the following [Actions](../../Player%20UI/Actions.md) option (the buzzer is played instead):

- `Skill`
- `Item`
- `Relay` 

This condition will also deny the confirmation of another player party member's choice when selecting who to relay to when handling the `Relay` choice for a `SelectPlayer` action. Effectively, it means not only the condition prevents the player party member from relaying, it also prevents them to be relayed to.

## [SetMaxOptions](../../Player%20UI/SetMaxOptions.md)
This condition causes the vine icons for these [Actions](../../Player%20UI/Actions.md) to render with the MainManager.`grayscale` material instead of MainManager.`spritedefaultunity`:

- `Skill`
- `Item`
- `Relay` 

## [ClearStatus](../Conditions%20methods/ClearStatus.md)
This condition is excluded from removal meaning it will remain even after calling this method.

## [UpdateAnim](../../Visual%20rendering/UpdateAnim.md)
For all player party members  with an `hp` above 4 with this condition and whose battleentity.`overrideanim` is false, their battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) may be set to AngryAnim which depends on the [animid](../../../Enums%20and%20IDs/AnimIDs.md): 10 (`Flustered`) for `Bee`, 5 (`Angry`) for `Beetle` and 102 for `Moth`. Check the method's documentation for the exact conditions.

## About enemy party members
The `BeetleTaunt` [skill](../../../Enums%20and%20IDs/Skills.md) allows to inflict this condition to an enemy party member, but it effectively doesn't do anything on its own for enemy party members because the effects described above only concerns the player party.

Instead, the skill itself has logic to affect enemy party members: it sets the `forceattack` field to `currentturn` (the player party member using the `BeetleTaunt` skill). This has the only effect of changing the logic of [GetRandomAvaliablePlayer](../Targetting/GetRandomAvaliablePlayer.md) to override its return to `forceattack` if `playerdata[forceattack].eatenby` is null (otherwise, it generates a random target ignoring the eaten player party member).

The overall effect is enemy party members using this method in their action logic will be forced to target whoever used the skill. It implies the conditions doesn't make them do that, only the skill's action logic does.
