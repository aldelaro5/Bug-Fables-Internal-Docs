# RefreshEntities
This method performs various tasks on all entities and [NPCControl](../Entities/NPCControl/NPCControl.md) to maintain their state consistency:

```cs
public static void RefreshEntities()
public static void RefreshEntities(bool onlyplayer)
public static void RefreshEntities(bool forceanim, bool refreshmap)
public static void RefreshEntities(bool forceanim, bool refreshmap, bool onlyplayer)
```
The name is ambiguous because it's hard to summarise everything it does, but as a general rule, this method is called when changes to entities needs to be reflected in their state. It is meant only to be called occasionally after performing changes that could affect the states of entities that this method regulates. It's best to check the tasks done by this method before concluding if it should be called. Notably, it forces a lot of EntityControl updates to happen such as [UpdateSprite](../Entities/EntityControl/Update%20process/UpdateSprite.md).

The parameters configure the refresh:

- `forceanim`: If true, any entity getting a refresh will have ForceAnim called on them so their current [animstate](../Entities/EntityControl/Animations/animstate.md) plays on their `anim`
- `refreshmap`: If true, several NPCControl refreshes happens and Clear is called on all TrailRenderer in the scene
- `onlyplayer`: If true, the method only refresh the states of `playerdata` overworld `entity`

For any overloads without certain parameters, their values all default to false.

The following sections are the detailed procedure this method does.

## `playerdata` refreshes
This section always happens. For any `playerdata` overworld `entity` that isn't null:

- Reset their `hitwall` to false
- Set their [animid](../Enums%20and%20IDs/AnimIDs.md) to their BattleData's animid (check the [player party](../General%20systems/Player%20party.md) documentation to learn more on the difference between the two)
- If the `forceanim` parameter is true, ForceAnim is called on the entity
- UpdateSpriteMat is called on the entity (sets their `sprite` material to MainManager.`spritemat` unless they are `hologram` where it's MainManager.`holosprite` instead)

After this section is done, if `onlyplayer` is true, the method immediately returns.

## Other EntityControl refreshes
This section only happens if `onlyplayer` is false. For any EntityControl in the scene (including the `playerdata` ones):

- Their `oldid` and `oldstate` are set to -1 which forces an [UpdateSprite](../Entities/EntityControl/Update%20process/UpdateSprite.md) to happen on the entity
- Their `emotioncooldown` is reset to 0.0 which will stop showing their emoticon
- Their `hitwall` is reset to false
- If their `height` is above 0.1 (there is a visual y offset), their `oldfly` is set to to the inverse of their `flyinganim` (This forces an UpdateSprite to happen, but the changes above already guaranteed that)
- If this is an [item entity](../Entities/EntityControl/Item%20entity.md), UpdateItem is called on them
- If the `forceanim` parameter is true, ForceAnim is called on the entity

After this section is done, if `refreshmap` is false, the method doesn't won't do anymore logic than the above.

## NPCControl refreshes
This section only happens if `onlyplayer` is false and `refreshmap` is true. For any EntityControl in the scene (including the `playerdata` ones), their `npcdata` gets the following happening to them (check the linked documentations to learn more about the conerned NPCControl logic):

- Some [Disguise behavior](../Entities/NPCControl/ActionBehaviors/Disguise.md) specific logic happens here
- Some [Dropplet](../Entities/NPCControl/ObjectTypes/Dropplet.md) and [PushRock](../Entities/NPCControl/ObjectTypes/PushRock.md) specific logic happens here
- Some [Geizer](../Entities/NPCControl/ObjectTypes/Geizer.md) specific logic happens here

Finally, all TrailRenderer in the scene gets Clear called on them.
