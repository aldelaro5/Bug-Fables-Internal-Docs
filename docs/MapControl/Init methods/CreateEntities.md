# CreateEntities
MapControl's Start contains a call to a method called CreateEntities which will initialise many [Entity](../../Entities/Entity.md) defined in preset data that are defined in [map entity data](../../TextAsset%20Data/Entity%20data.md#map-entity-data).

The map entity data directory contains entity data that applies to each specific [Maps](../../Enums%20and%20IDs/Maps.md) hence why this creation process happens on MapControl. The [EntityControl Creation](../../Entities/EntityControl/EntityControl%20Creation.md) process is exactly the same, but the different fields from map data augments this process right before the entity is fully created and EntityControl's [Start](../../Entities/EntityControl/Start.md) occurs later on the start of the next frame.

Exclusive to this scheme of creating entities is making them map entities with [NPCControl](../../Entities/NPCControl/NPCControl.md) as the fields loaded includes both [EntityControl](../../Entities/EntityControl/EntityControl.md) and [NPCControl](../../Entities/NPCControl/NPCControl.md) fields. The NPCControl ends up in the `npcdata` field of the EntityControl. All the data ends up in the `entities` array of the MapControl which can be used for tracking and the index of each is also assigned to the `mapid` field of the NPCControl. They are loaded in the order they are defined in TextAsset data which means the 0 indexed position can be used as an id. This is frequently done as part of many [SetText](../../SetText/SetText.md) commands or NPCControl operations that affects other NPCControl.

This page only describe the behaviors outside of loading the data as there are some notes worth to talk about (most of the method's logic involves loading the [map entity data](../../TextAsset%20Data/Entity%20data.md#map-entity-data) so check that documentation to learn more about the loading).

## Debug feature
If the name of an entity contains the string `debug`, the entire creation process is skipped for this entity unless MainManager's `debugenalbed` is true. This particular field is never set explicitly: it must be set externally and outside of the game's control because it is not possible to enable it through normal means. It should be noted that no entities defined in the game uses this feature, but it still remains functional.

## Position start
If the player exists by the time the entities are created, the entity's default position is set to be 2.0 above the player. This is before setting their actual position from map entity data.

## The `ROT` modifier
This modifier also changes the way angles are loaded on the entity. Normally, this comes from the entity data, but if this modifier is present, it will instead start a LateAngle on the entity with the entity data's angles in 0.25 seconds of yield time.

## DoorSameMap logic
There is some [DoorSameMap](../../Entities/NPCControl/ObjectTypes/DoorSameMap.md) specific logic here. More information can be found at the DoorSameMap documentation.

## Shop logic
This is where the [shop system](../../Entities/NPCControl/Shop%20system.md) structure is built. More information can be found at the shop system documentation.
