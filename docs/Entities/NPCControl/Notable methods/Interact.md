# Interact
This public method is one of the few that is fully intended to be called externally. It allows to have an NPCControl perform an interaction and most of its logic is defined by its `interacttype` which is of type [Interaction](../Interaction.md). More commonly, this is used by PlayerControl whenever the player interacts with the NPCControl, but it can be called externally to simulate the same effects then the interaction.

The method takes a string argument named `args`, but only a few `Interaction` uses it.

There is some common logic that precedes any `Interaction` specific logic and it goes as follows:
- [StopForceMove](../EntityControl/EntityControl%20Methods.md#StopForceMove) is called on the entity
- [StopMoving](../EntityControl/EntityControl%20Methods.md#StopMoving) is called on the player entity
- CancelAction is called on the player

After, the logic is specific to each `Interaction`. Consult each's documentation to learn more.