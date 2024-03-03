# StartBattle
This is a public coroutine that serves as the only way for the game to start a battle.

```cs
public static IEnumerator StartBattle(int[] enemyids, int stageid, int adv, string music, NPCControl calledfrom, bool canescape)
```

## Parameters

- `enemyids`: The list of [enemies](../Enums%20and%20IDs/Enemies.md) id that the player party will be against. If it contains more than 4 elements, all the ones after the 4th one will be placed in `extraeenmies` and the fight will be against the first 4 for now by setting `enemydata` to them. This is subject to be overriden by EnemyCheck during the start process
- `stageid`: The [BattleMaps](../Enums%20and%20IDs/BattleMaps.md) to use for the battle, will be overriden to the ice version if the calledfrom.entity is `inice` and the stageid is `SandCastle` or `SandCastleDark`. If -1 is sent, the stage is determined by instance.`battlestage` which the MapControl initialised on its Start
- `adv`: Decides whose party has the advantage. This will be set to battle.`sadv`. Only the value 3 does anything and it's to give the enemy party a `firststrike`. Other values are allowed, but do not change anything
- `music`: The name of the music to use for the battle. This is ignored if map.`nobattlemusic` is true. If it's null, the music is determined automatically based on the [BattleMaps](../Enums%20and%20IDs/BattleMaps.md) and some conditions
- `calledfrom`: The [Enemy](../Entities/NPCControl/Enemy.md) NPCControl that caused this battle if any. If it's null, it means no NPCControl caused it and the battle was started manually. This will be set to battle.`caller`
- `canescape`: Whether the player is allowed to flee or not. This will be set to battle.`canflee`

## Procedure
The procedure for starting a battle is very complex and there's multiple logic to it which is sequentially performed. The caller is able to interupt it at 2 places to configure the battle further: one early on by setting MainManager.`haltbattleload` to true and one later on by yielding on battle.`halfload` becoming true.

The phases will be documented in their own page where they are defined on the boundaries of those interuption points. Here are the phases in order and a link to their details:

- [Pre haltbattleload](StartBattle%20phases/Pre%20haltbattleload.md)
- [Post haltbattleload](StartBattle%20phases/Post%20haltbattleload.md)
- [Post halfload](StartBattle%20phases/Post%20halfload.md)
