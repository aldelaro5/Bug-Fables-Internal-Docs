# RefreshEntities
This method performs some reset on some entity fields. It belongs to MainManager.

```cs
public static void RefreshEntities(bool forceanim, bool refreshmap, bool onlyplayer)
```

## Parameters:

- `forceanim`: Tells if FroceAnim will be caolled on each entity which will call Play on the entity's `anim` using its the current [animstate](../../Entities/EntityControl/Animations/animstate.md) (`f` is appended if the entity's `height` is aboe 0.1 and it has the same meaning than its normal argument)
- `refreshmap`: Tells if the [NPCControl](../../Entities/NPCControl/NPCControl.md) specific logics will apply
- `onlyplayer`: Tells if only the `playerdata` entities part will have their fields reset

There are 3 overloads: one where `onlyplayer` is ommited and false is sent, one without parameter where all the parameters will have false sent and one where the only parameter is `onlyplayer` and the other 2 will have false sent.

## Procedure

- All `playerdata` entities have their `hitwall` set to false and their [animid](../../Enums%20and%20IDs/AnimIDs.md) set to their corresponding BattleData.`animid`. If forceanim is true, ForceAnim is called which will call Play on the entity's `anim` using its the current [animstate](../../Entities/EntityControl/Animations/animstate.md) (`f` is appended if the entity's `height` is aboe 0.1 and it has the same meaning than its normal argument)
- We return immediately if `onlyplayer` is true
- All [EntityControl](../../Entities/EntityControl/EntityControl.md) have the following happen:
  - `oldid` is set to -1
  - `oldstate` is set to -1
  - `emoticoncooldown` is set to 0.0
  - `hitwall` is set to false
  - If `height` is above 0.1, `oldfly` is set to `flyinganim`
  - If it's an [item entity](../../Entities/EntityControl/Item%20entity.md), [UpdateItem](../../Entities/EntityControl/Update%20process/UpdateItem.md) is called
  - If forceanim is true, ForceAnim is called which will call Play on the entity's `anim` using its the current [animstate](../../Entities/EntityControl/Animations/animstate.md) (`f` is appended if the entity's `height` is aboe 0.1 and it has the same meaning than its normal argument)
  - If refreshmap is true and the entity has an `npcdata`, further logic are done about the corresponding [NPCControl](../../Entities/NPCControl/NPCControl.md) for the following:
    - [Disguise](../../Entities/NPCControl/ActionBehaviors/Disguise.md) and its derivatives
    - [Dropplet](../../Entities/NPCControl/ObjectTypes/Dropplet.md)
    - [PushRock](../../Entities/NPCControl/ObjectTypes/PushRock.md)
    - [Geizer](../../Entities/NPCControl/ObjectTypes/Geizer.md)
- If refreshmap is true and the map exists, all TrailRenderer has Clear called on them which clears all the wind trails currently active