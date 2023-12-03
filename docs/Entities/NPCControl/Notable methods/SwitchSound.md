# SwitchSound
This is a private method in NPCControl that aims to play the sound of turning on or off a wide variety of switch like objects. It receives a bool to indicate if the object is being actuated or deactuated.

Nothing happens if the `startlife` hasn't reached at least 15.0 frames.

In all cases, the `Button` sound is played on the entity with 1.0 volume, but the pitch depends on the actuation. If we are actuating, it's 1.0 pitch and 0.5 if we are deactuating.

There's a special case if the entity.`originalid` is the `AncientPressurePlate`, `SwitchCrystal` or `BigCrystalSwitch` [animid](../../../Enums%20and%20IDs/AnimIDs.md). In that case another sound is played being a `Glow` sound using the same pitch variation as the other one, but the volume is the entity sound distance.