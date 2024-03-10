# SummonEnemy
This is a coroutine that allows to create an [enemy](../../../Enums%20and%20IDs/Enemies.md) and to add it to `enemydata` during the battle.

There are 2 overloads:

(1)
```cs
private IEnumerator SummonEnemy(SummonType type, int enemyid, Vector3 position)
```
This one ends end starting (2) with cantmove as false followed by a frame yield.

(2)
```cs
private IEnumerator SummonEnemy(SummonType type, int enemyid, Vector3 position, bool cantmove)
```

## Parameters

- `type`: The type of summon to use which describes the method the enemy should appear
- `enemyid`: The [enemy](../../../Enums%20and%20IDs/Enemies.md) id to summon
- `position`: The position the enemy should appear in battle
- `cantmove`: If true, the starting `cantmove` value of the new enemy will be 1 (meaning one actor turn will need to pass for the enemy to act)

## SummonType
This is an enum that tells the behavior to have when summoning in this method (see the procedure for details):

|Value|Name|Description|
|-----|----|-----------|
|0|FromGround|The screen is shook twice followed by the enemy appears from the ground with a final `startscale` of (1.15, 1.15, 1.15)|
|1|Offscreen|The enemy appears by a [movetowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) call from the right offscreen|
|2|OffscreenNoAnim|The same than `Offscreen`, but the position is lerped instead of a [movetowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) call|
|3|None|No specific summoning logic occurs|
|4|FromGroundInstant|An alias of `FromGround`|
|5|Raise|The same than `OffscreenNoAnim`, but the enemy appears from the bottom of the screen|
|6|FromGroundKeepScale|The same than `FromGround`, but the final `startscale` is the one from [endata](../../../TextAsset%20Data/Entity%20data.md#animid-data)|
|7|Tablet|The position to summon is overriden to (50.0, 50.0, 0.0)|

## Procedure
There's 3 sections to this coroutine: 

- `SummonType` pre add logic
- Adding the enemy
- `SummonType` post add logic
- Last adjustements

### `SummonType` pre logic
The logic depends on the sent type:

#### `FromGround` / `FromGroundKeepScale`

- ShakeScreen is called for 0.5 seconds
- 0.5 seconds are yielded
- ShakeScreen is called again with an ammount of (0.2, 0.2, 0.2) for 0.1 seconds

### `Offscreen` / `OffscreenNoAnim`

- The x/y position is overriden to 20.0 and 0.0 respectively

### `Raise`

- The y position is overriden to position.y - 20.0

### `Tablet`

- The position is overriden to (50.0, 50.0, 0.0)

### Adding the enemy
[AddNewEnemy](AddNewEnemy.md) is called with the enemyid and the position (or the overriden one if applicable). The [EntityControl](../../../Entities/EntityControl/EntityControl.md) of the new enemy is tracked locally.

### `SummonType` post add logic
The logic depends on the sent type (all position refers to the original sent value, not the overriden one unless stated otherwise. Additionally, the entity reffered to is the new enemy entity stored locally):

#### `Tablet`

- `TabletSpawn` particles are played at the position + (0.0, entity.`height`, 0.0) for 0.5 seconds
- 0.5 seconds are yielded
- The `Woosh6` sound is played
- DeathSmoke particles are played at the position + (0.0, entity.`height`, 0.0)
- ShakeScreen is called with an ammount of 0.1 for 0.5 seconds with dontreset
- The position is set to the sent position
- SlowSpinStop is called on the entity with a spinammount of (0.0, 30.0, 0.0) for 60.0 frames
- A second is yielded

#### `FromGround` / `FromGroundInstant`

- DeathSmoke paricles are played at the entity position
- entity.`startscale` is zeroed out
- entity.`sprite` local scale is set to entity.`startscale`
- A frame is yielded
- If the entity has a [model](../../../Entities/EntityControl/Notable%20methods/AddModel.md), entity.`model` local scale is set to entity.`modelscale`
- If enemyid isn't a `PitcherFlytrap` [enemy](../../../Enums%20and%20IDs/Enemies.md), entity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 101
- entity.`spin` is set to (0.0, 20.0, 0.0)
- During the course of 30.0 frames tracked with a local frame counter (incremented by the game's frametime):
    - entity.`startscale` is set to a lerp from Vector3.zero to (1.15, 1.15, 1.15) with a factor of the ratio of the amount of frames cumulated over 30.0 frames
    - entity.`sprite` local scale is set to entity.`localscale`
    - The local framecounter is advanced
    - A frame is yielded
- entity.`startscale` is set to (1.15, 1.15, 1.15)
- entity.`spin` is zeroed out
- 0.5 seconds are yielded
- entity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 0 (`Idle`)

#### `FromGroundKeepScale`
The same than `FromGround` / `FromGroundInstant`, but instead of having a final scale of (1.15, 1.15, 1.15) grown smoothly, the target scale is the `startscale` from [endata](../../../TextAsset%20Data/Entity%20data.md#animid-data)

#### `Offscreen`

- A frame is yielded
- entity.`rigid` has its gravity disabled
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) is called on the entity to the sent position with a 2.0 multiplier
- All frames are yielded while the entity is in a `forcemove`
- entity.`rigid` has its gravity enabled
- entity position is set to the sent position

#### `OffscreenNoAnim` / `Raise`

- A frame is yielded
- entity.`rigid` has its gravity disabled
- During the course of 101.0 frames tracked with a local frame counter (incremented by the game's frametime):
    - entity's position is set to a lerp from the position (or the overriden one if applicable) to the sent position (not the overriden one) with a factor of the ratio of the amount of frames cumulated over 101.0 frames
    - The local framecounter is advanced
    - A frame is yielded
- entity.`rigid` has its gravity enabled
- entity's position is set to the sent position

### Last adjustements

- If the sent cantmove is true, `enemydata[lastaddedid].cantmove` is set to 1 (one actor turn will need to pass for the enemy party member to act)
- If `summonnewenemy` is false, `checkingdead` is set to null
- `summonnewenemy` is set to false
