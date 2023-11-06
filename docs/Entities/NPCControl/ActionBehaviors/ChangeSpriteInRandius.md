# ChangeSpriteInRandius
When the player is present, the entity [animstate](../../EntityControl/Animations/animstate.md) is set to `actionfrequency[0]` when the distance between this and the player is less than the `wanderradius`, it is set to `actionfrequency[1]` otherwise.

## Frequency meaning
Each of the 2 frequencies corresponds to an [animstate](../../EntityControl/Animations/animstate.md) to set the entity depending on the distance between it and the player in regards to the `wanderradius`.

NOTE: this behavior does NOT read the frequency passed in DoBehavior

## DoBehavior
If the player isn't present, this does nothing.

Otherwise, if the distance between this entity and the player is less than the `wanderradius`, the entity [animstate](../../EntityControl/Animations/animstate.md) is set to `actionfrequency[0]`. it is set to `actionfrequency[1]` otherwise.