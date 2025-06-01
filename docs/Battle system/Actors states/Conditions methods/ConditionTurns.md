# ConditionTurns
This is a method that belongs to MainManager:

```cs
public static int ConditionTurns(int playerid, int turns)
```
Simply returns `turns`. `playerid` is ignored.

As odd as this method seems, it is used once: during [FrigidCoffin](../../Player%20actions/Skills/FrigidCoffin.md)'s action logic when inflicting the [Freeze](../BattleCondition/Freeze.md) condition for 2 main turns. It's used to get that "2" main turns amount, but because it just returns 2 in that case, it didn't ended up doing anything useful.
