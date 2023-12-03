# NPCControl
NPCControl is a component that defines the behaviors of any map [Entities](../Entity.md). This only includes entities loaded from [map entity data](../../TextAsset%20Data/Entity%20data.md#map-entity-data) and all entities created for the purpose of a [shop system](Shop%20system.md). Its name is a missnomer because it doesn't just manage NPCs, but also objects and enemies so it should be viewed as "MapEntityControl".

Unlike [EntityControl](../EntityControl/EntityControl.md), this component exposes very little to be accessed externally with some exceptions, but most of these exceptions covers very narrow cases such as simply loading the field when map loading happens. One notable exception is [Interact](Notable%20methods/Interact.md) which offers the ability to provoke an interaction with the NPCControl.

Most of this component's logic is into its startup, update and colliders handling lifecycle. It is mainly dictated by 4 elements:

- The [NPCType](NPCType.md)
- The [ObjectTypes](Object.md#objecttypes) (when the NPCType is [Object](Object.md))
- The default and `inrange` [ActionBehaviors](ActionBehaviors.md)
- The [Interaction](Interaction.md)
