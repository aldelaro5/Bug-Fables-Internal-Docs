# GetAlivePlayerAmmount
This is a method on MainManager that is used a lot in the battle system to tell how many player party members are considered alive:

```cs
public static int GetAlivePlayerAmmount()
```
Returns the amount of `playerdata` whose `hp` are above 0 and aren't `eatenby`.

This is different than [GetFreePlayerAmmount](GetFreePlayerAmmount.md) because that one tells how many player party members can act. GetAlivePlayerAmmount tells how many are considered alive which means when this returns 0, the battle system should proceed to end the battle in a loss.
