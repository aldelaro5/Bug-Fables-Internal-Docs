# AddNewEnemy
This is a method that will initialise a new enemy and add it to `enemydata`.

There are 3 overloads available:

(1)
```cs
private Transform AddNewEnemy(int id, Vector3 position)
```

(2)
```cs
private Transform AddNewEnemy(int id, Vector3 position, SpawnAnim animation)
```
Ends up doing (1) if animation is `None` (0)

(3)
```cs
private Transform AddNewEnemy(int id, EntityControl entity)
```
NOTE: This overload is unused under normal gameplay. It is similar to (1), but it will not be documented due to older logic

## Parameters

- `id`: The [enemy](../../../Enums%20and%20IDs/Enemies.md) id to add
- `position`: Where the enemy should initialy be spawned
- `animation`: If it's not `None` (0), [NewEnemy](NewEnemy.md) will be called instead followed by a frame yield (the method ends up calling (1), but it has more logic afterwards). If it's `None` (0), it ends up being the same as (1)

## Procedure
Since all overloads ends up doing (1), this is the procedure that will be documented. Null is returned immediately if `enemydata` already has 4 elements or more (it's full, this doesn't handle `extraenemies`).

- A new `BattleData` is obtained from [GetEnemyData](../../../TextAsset%20Data/Enemies%20data.md#getenemydata) with createentity. From now own, the enemy refered to is the new one
- battleentity's position is set to the sent position
- battleentity.`battleid` is set to the length of `enemydata`
- battleentity is childed to `battlemap`
- battleentity.`hologram` is set to the value of [flags](../../../Flags%20arrays/flags.md) 162 (Using the B.O.S.S system or during a Cave of Trials session)
- If the current [map](../../../Enums%20and%20IDs/Maps.md) is `CaveOfTrials` and the matching [bestiary entry](../../../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md) has a seen counter of 0:
    - battleentity.`cotunknown` is set to true
    - ForceCOT is called on the battleentity
    - RefreshCOT is invoked in 0.1 seconds on the battleentity
- The new enemy is added to `enemydata` at the end
- If `animid` (the [enemy](../../../Enums%20and%20IDs/Enemies.md) id) is less than the length of instance.`enemyencounter` (normally 256) and battleentity.`cotunknown` is false, the view count of the matching [bestiary entry](../../../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md) is incremented
- `lastaddedid` is set to the the `enemydata` index matching the enemy that was just added
- The enemy's battleentity.transform is returned
