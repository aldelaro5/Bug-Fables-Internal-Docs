# Entity

An entity in Bug Fables is an object that could be seen as the building blocks of the game. It features common logic used throughout the game and allows to have complex runtime behaviors. In other words, it's what allows the game to have any kind of interactions whether it's movement, enemy encounters or loading zones to name a few. It is made of 2 components, one of them being optional:

* [EntityControl](EntityControl/EntityControl.md): This defines what an entity can do and defines the most common aspects of all entities. It can be viewed as a component that gives utilities and modifiers for an object to have. 
* [NPCControl](NPCControl/NPCControl.md): This leverages the utilities and modifiers defined in [EntityControl](EntityControl/EntityControl.md) to implement more complex behaviors and logic for entities predefined in maps. Any other entities created manually outside of exceptional cases do not use an [NPCControl](NPCControl/NPCControl.md). The component is stored in the `npcdata` field of [EntityControl](EntityControl/EntityControl.md).

## Differences between an entity and an NPC

Due to these components having a clear delimitation, it gives a new definition to the word NPC for this game. An NPC is an entity that was predefined in a map and therefore, it will be created with an [NPCControl](NPCControl/NPCControl.md). In other words, all NPC are entities, but not all entities are NPC.

An NPC has to be created from [CreateEntities](EntityControl/CreateEntities.md) of MapControl through \[\[TextAsset Data/Entity data#`entitydata` directory|Entity data\]\] while an entity can be created manually through the standard [EntityControl Creation > Creating an entity](EntityControl/EntityControl%20Creation.md#creating-an-entity) process. While it is possible to make any entities an NPC by assigning the `npcdata` field to a new NPCControl, this isn't supported outside of very exceptional cases and isn't encouraged in practice.
