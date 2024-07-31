# Inked
A condition that prevents a player party member from using a [skill](../../../Enums%20and%20IDs/Skills.md) that may be inflicted in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) if property is `Ink` or `InkOnBlock` (without a resistance field) featuring a `Prefabs/Particles/InkDrip` as a `firepart`.

## [GetChoiceInput](../../Player%20UI/GetChoiceInput.md) 
This condition prevents the `Skill` [Actions](../../Player%20UI/Actions.md) option (the buzzer is played instead).

## [SetMaxOptions](../../Player%20UI/SetMaxOptions.md)
This condition causes the vine icons for the `Skill` [Actions](../../Player%20UI/Actions.md) to render with the MainManager.`grayscale` material instead of MainManager.`spritedefaultunity`.

## [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md)
This condition may be inflicted if the property is `Ink` or `InkOnBlock`. This means it's also supported by the `StatusMirror` [medal](../../../Enums%20and%20IDs/Medal.md). It does not use a resistance field, check the method's documentation to learn more on the conditions needed for the infliction to work.

## [UpdateEntities](../../Visual%20rendering/UpdateEntities.md)
This condition will have the battleentity.`sprite`.material.color to be set to a lerp from the existing one to 0x7200D8 (bright purple) with a factor of 1/5 of the frametime. Check the documentation of the method to learn more about how the alpha channel is determined.

This condition includes the initialisation of the actor's `firepart` as a new instance of `Prefabs/Particles/InkDrip` childed to the `sprite` with a `DelAftBtl` tag and a local position of Vector3.up.

## About enemy party members
This condition effectively doesn't do anything on its own for enemy party members because the effects described above only concerns the player party.

It is up to the individual [enemy action](../../Enemy%20actions/Enemy%20actions.md) to implement logic to this condition which may include changes to their move selection process.
