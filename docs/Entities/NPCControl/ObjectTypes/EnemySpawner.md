# EnemySpawner
An enemy spawner attached to another [Enemy](../NPCType.md#enemy) entity that will continuously respawn in a given circle range whenever it dies after a certain amount of time. The map entity id of the enemy, the time interval to wait before respawn and the range at which the enemy will spawn is configurable with the option to spawn the enemy immediately on the first Update cycle.

## Data Arrays
- `data[0]`: The map entity id of the NPCControl that refers to the [Enemy](../NPCType.md#enemy) to spawn
- `data[1]`: If it's 1, the spawner spawns the enemy on the first active Update cycle instead of having to wait the `data[4]` cooldown expires
- `data[2]`: UNUSED
- `data[3]`: UNUSED
- `data[4]`: The time in frames to wait before the spawner spawns the enemy
- `data[5]`: OVERRIDEN (the original entity.[animid](../../../Enums%20and%20IDs/AnimIDs.md))
- `vectordata[0]`: The center position of the spawning range
- `vectordata[1]`: The range of the spawning. This says the random offset to apply to `vectordata[0]` to apply as the spawn position which is from -`vectordata[1]` to `vectordata[1]` with + (0.0, 0.5, 0.0) being added to it

## Setup
- The entity.`rigid` is put in kinematic mode
- The gameObject's isStatic is set to true
- `actioncooldown` is set to `data[4]` unless `data[1]` is 1 where it's set to 0.0 instead
- After saving the old value in `data[5]`, the entity's `animid` is set to -1 (none)
- entity.`sprite` is disabled
- entity.`ccol` is disabled
- `alwaysactive` is set to true

## Update
There are 2 cases here. If the `actioncooldown` isn't expired and the `spawned` enemey doesn't exist or its entity `iskill`, the `actioncooldown` is decremented by the game's frametime which ends this update.

If the cooldown has expired, then this update will only occur if the `spawned` doesn't exist or its entity `iskill`. This means that we should attempt to spawn the enemy. Otherwise, all update cycles are ignored until this becomes true.

If the `spawned` enemy still doesn't exist, it is set to the NPCControl of the `Enemy` whose map entity id is `data[0]`.

Here, as long as the `spawned`'s entity exists, [RespawnEnemy](../RespawnEnemy.md) is called with the `spawned` enemy at the position `vectordata[0]` + RandomVector(`vectodata[1]`) + (0.0, 0.5, 0.0). The random vector portion is from -`vectordata[1]` to `vectordata[1]`.

Finally, the `actioncooldown` is reset to `data[4]`.
