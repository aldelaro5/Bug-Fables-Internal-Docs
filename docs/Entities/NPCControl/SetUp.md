# Setup
This method is only called during [Start](Start.md) and it contains most of the logic of the startup process. The logic depends on the [NPCType](NPCType.md), [ObjectTypes](Object.md#objecttypes), [ActionBehaviors](ActionBehaviors.md) and [Interaction](Interaction.md). Consult each's documentation to learn more as only the end part of the method is shared.

## Common startup
At the end of the method, this phase happens regardless of the logic performed prior.

CheckHidden is invoked in 1 second which setups the logic to check if the NPCControl should be considered a hidden item. This happens if [HasHiddenItem](Notable%20methods/HasHiddenItem.md) returns true and CheckIfCanExist returns false according to the `requires`, `limit` and `regional`. This will cause the Detector [medal](../../Enums%20and%20IDs/Medal.md) to beep and the correct emoticon (id 4) to be set on the player entity for 100.0 frames, but this part is done by MapControl's LateUpdate.

After, the entity.`rigid` gets frozen on all constraints if the entity.`fixedentity` or `freezeconstraints` is true.

Finally, the entity.`initialcolliderdata` is set back to (entity.`ccol.height`, entity.`ccol.radius`).
