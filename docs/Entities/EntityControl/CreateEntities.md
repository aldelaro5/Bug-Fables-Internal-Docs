# MapControl's CreateEntities

MapControl's Start contains a call to a method called CreateEntities which will initialise many [Entity](../Entity.md) defined in preset data that are defined in [entitydata directory](../../TextAsset%20Data/Entity%20data.md#`entitydata`%20directory). This directory contains entity data that applies to each specific [Maps](../../Enums%20and%20IDs/Maps.md) hence why this creation process happens on MapControl. The [EntityControl Creation](EntityControl%20Creation.md) process is exactly the same, but the different fields from map data augments this process right before the entity is fully created and [Start](Start.md) occurs later on the start of the next frame. Exclusive to this scheme of creating entities is making them NPC with [NPCControl](../NPCControl/NPCControl.md) as the fields loaded includes both [EntityControl](EntityControl.md) and [NPCControl](../NPCControl/NPCControl.md) All the data ends up in the `entities` array of the MapControl which can be used for tracking. They are loaded in the order they are defined in TextAsset data which means the 0 indexed position can be used as an id. This is frequently done as part of many [SetText](../../SetText/SetText.md) commands.

This page only describe the behaviors outside of loading the data as there are some notes worth to talk about.

## Debug feature

If the name of an entity contains the string `debug`, the entire creation process is skipped for this entity unless MainManager's `debugenalbed` is true. This particular field is never set explicitly: it must be set externally and outside of the game's control because it is not possible to enable it through normal means. It should be noted that no entities defined in the game uses this feature, but it still remains functional.

## Position start

If the player is loaded by the time the entities are created, their default position is 2.0 above the player.

## The `ROT` modifier

This modifier also changes the way angles are loaded on the entity. Normally, this comes from the entity data, but if this modifier is present, it will instead start a LateAngle on the entity with the entity data's angles in 0.25 seconds of yield time.

## DoorSameMap logic

TODO: come back to this when [NPCControl](../NPCControl/NPCControl.md) is documented

## Shop and ShopKeeper logic

TODO: come back to this when [NPCControl](../NPCControl/NPCControl.md) is documented
