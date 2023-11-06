# EnemySpawner
An enemy spawner attached to another `Enemy` entity that will continuously respawn in a given circle range whenever it dies after a certain amount of time. The map entity id of the `Enemy`, the time interval between checks and the range at which the enemy will spawn is configurable.

## Data Arrays
- `data[0]`: The map entity id of the NPCControl that refers to the `Enemy` to spawn.
- `data[1]`: If it's 1, `actioncooldown` is set to 0.0 and if not, it's set to `data[4]`
- `data[2]`: ???
- `data[3]`: ???
- `data[4]`: The `actioncooldown` if `data[1]` isn't 1
- `data[5]`: ??? (overriden to the entity's [animid](../../../Enums%20and%20IDs/AnimIDs.md))
- `vectordata[0]`: The center position of the range
- `vectordata[1]`: The radius of the range. This says the random offset to apply to `vectordata[0]` to apply which is from -`vectordata[1]` to `vectordata[1]`.

## Setup
The object's isStatic is set to true followed by some adjustements performed on the entity:
- The `rigid` is put in kinematic mode
- After saving the old value in `data[5]`, the entity's `animid` is set to -1 (none)
- The `sprite` is disabled
- The `ccol` is disabled
- `alwaysactive` is set to true

## Update
There are 2 cases here. If the `actioncooldown` isn't expired and the `spawned` doesn't exist or its entity `iskill`, the `actioncooldown` is decremented by the game's frametime which ends this update.

If the cooldown has expired, then this update will only occur if the `spawned` doesn't exist or its entity `iskill`. This means that we should attempt to spawn the enemy. Otherwise, all update cycles are ignored until this becomes true.

If the `spawned` is still null by then, it is set to the NPCControl of the `Enemy` whose map entity id is `data[0]`.

Here, as long as the `spawned`'s entity isn't null, [RespawnEnemy](../RespawnEnemy.md) is called with the `spawned` enemy at the position `vectordata[0]` + RandomVector(`vectodata[1]`) + (0.0, 0.5, 0.0). The random vector portion is from -`vectordata[1]` to `vectordata[1]`.

Finally, the `actioncooldown` is reset to `data[4]`.
