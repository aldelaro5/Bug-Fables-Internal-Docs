# Start
The event does nothing if it's a `dummy`.

The entity.`ccol` and `scol` are initialised according to their [NPCType](NPCType.md).

From there, [SetUp](SetUp.md) is called which contains most of the startup logic.

After, a couple of specific logic occurs:

- The tag is set according to the [NPCType](NPCType.md)
- The entity.`rigid` mass is changed according to the [NPCType](NPCType.md) except if it's an [item entity](../EntityControl/Item%20entity.md) where it remains 1.0
- If applicable, the `disguiseobj` is initialised according to the [ActionBehaviors](ActionBehaviors.md)
- The entire gameObject gets disabled if the `requires` and `limit` contain a violation. It also gets disabled when the entity.`hideinside` is true while the `insideid` does not match the current one
- If the `NGF` [Modifier](../EntityControl/Modifiers.md) applies, [GravityFix](Notable%20methods/GravityFix.md) is started