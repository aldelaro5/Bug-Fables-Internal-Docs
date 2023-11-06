# NPCControl

NPCControl is a component that defines the behaviors of any [Entity](../Entity.md) predefined in a map. Such an entity can be called a map entity, but not necessarily an NPC as NPCControl also deals with enemies and objects.

Unlike [EntityControl](../EntityControl/EntityControl.md), this component exposes very little to be accessed externally with some exceptions, but most of these exceptions covers very narrow cases such as simply loading the field when map loading happens. One notable exception is [Interact](Interact.md) which offers the ability to provoke an interaction with the NPCControl.

Most of this component's logic is into its startup, update and colliders handling lifecycle. It is mainly dictated by 4 elements:
- The [NPCType](NPCType.md)
- The [ObjectTypes](ObjectTypes.md) (when the NPCType is `Object`)
- The default and `inrange` [ActionBehaviors](ActionBehaviors.md)
- The [Interaction](Interaction.md) assigned
