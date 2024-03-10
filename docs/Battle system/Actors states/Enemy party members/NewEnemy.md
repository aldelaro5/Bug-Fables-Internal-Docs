# NewEnemy
This coroutine is a wrapper around [AddNewEnemy](AddNewEnemy.md) in syntax (1) that ends up doing more spawn logic after.

```cs
private IEnumerator NewEnemy(int id, Vector3 pos, SpawnAnim animation)
```

## Parameters

- `id`: The [enemy](../../../Enums%20and%20IDs/Enemies.md) id to add
- `position`: Where the enemy should initialy be spawned
- `animation`: If it's not `None` (1 or 2), the spawn animation to use for the new enemy. If it's `None` (0), the coroutine only calls [AddNewEnemy](AddNewEnemy.md) and set `checkingdead` to null

## Procedure
[AddNewEnemy](AddNewEnemy.md) is called with the id and position and store the entity locally.

If animation is `None` (0), the coroutine is done as it only sets `checkingdead` to null before ending

Otherwise, the following occurs (the entity refered to is the new enemy just added):

- The `Charge` sound is played
- If entity.`cotunknown` is true:
    - ForceCOT is called on the entity
    - `spritebasecolor` is set to be pure black at 50% opacity
- The entity local scale is zeroed out
- `spin` is set to (0.0, 25.0, 0.0)
- During the course of 60.0 frames tracked with a local frame counter (incremented by the game's frametime):
    - entity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 11 (`Hurt`)
    - entity's local scale is set to a lerp from Vector3.zero to a target that depends on the animation with a factor of the ratio of the amount of frames cumulated over 60.0 frames. The target is Vector3.one for an animation of `Seed` (1) and (0.75, 0.75, 0.75) for `SeedMini` (2)
    - The local framecounter is advanced
    - A frame is yielded
- `startscale` is set to Vector3.one
- `sprite` local scale is set to the same target used for the lerping above
- `spin` is zeroed out
- [animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 0 (`Idle`)
- If animation is `SeedMini` (2), `hpbar` local scale is increased by (0.25, 0.25, 0.25)
- `tempslot` is set to the last `enemydata`
- `checkingdead` is set to null and the coroutine ends
