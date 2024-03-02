# Sticky
A condition that prevents a player party member from using an [item](../../../Enums%20and%20IDs/Items.md) that may be inflicted in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) if property is `Sticky` (without a resistance field) featuring a `Prefabs/Particles/StickyDrip` as a `firepart`.

## [GetChoiceInput](../../Player%20UI/GetChoiceInput.md) 
This condition prevents the `Item` [Actions](../../Player%20UI/Actions.md) option (the buzzer is played instead).

## [SetMaxOptions](../../Player%20UI/SetMaxOptions.md)
This condition causes the vine icons for the `Item` [Actions](../../Player%20UI/Actions.md) to render with the MainManager.`grayscale` material instead of MainManager.`spritedefaultunity`.

## [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md)
This condition may be inflicted if the property is `Sticky`. This means it's also supported by the `StatusMirror` [medal](../../../Enums%20and%20IDs/Medal.md). It does not use a resistance field, check the method's documentation to learn more on the conditions needed for the infliction to work.

## [UpdateEntities](../../Visual%20rendering/UpdateEntities.md)
This condition includes the initialisation of the actor's `firepart` as a new instance of `Prefabs/Particles/StickyDrip` childed to the `sprite` with a `DelAftBtl` tag and a local position of Vector3.up.
