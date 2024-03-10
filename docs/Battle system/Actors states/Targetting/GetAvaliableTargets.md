# GetAvaliableTargets
This method updates `avaliabletargets` and refreshes the enemy positions when applicable

```cs
private void GetAvaliableTargets(bool onlyground, bool onlyfront, int attackid, bool excludeunderground)
```

## Parameters

- `onlyground`: Only include enemy party members with a [position](../BattlePosition.md) of `Ground` (or `Underground` if excludeunderground is false)
- `onlyfront`: Only the first eligible enemy party member will be included
- `attackid`: The [skill](../../../Enums%20and%20IDs/Skills.md) id or a negative number denoting a player party member's basic attack (-1 is `Bee`'s, -2 is `Beetle`'s and -3 is `Moth`'s)
- `excludeunderground`: Excludes all enemies with a [position](../BattlePosition.md) of `Underground` when `onlyground` is true. There is an overload without this parameter that sends false

## Procedure

There is a simpler case that is checked first.

If [currentaction](../../Player%20UI/Pick.md) is `SkillList` and the [AttackArea](../../Player%20UI/AttackArea.md) of the [skill](../../../Enums%20and%20IDs/Skills.md) from [skilldata](../../../TextAsset%20Data/Skills%20data.md#skilldata) is `AllParty` or `SingleAlly`, `availabletargets` is set to `playerdata` which ends the method.

Otherwise:

- RefreshEnemyPos is called which checks all enemy party members whose `hp` is above 0, whose [cantfall](../Enemy%20features.md#cantfall) is false and whose [position](../BattlePosition.md) is `Ground` or `Flying`. If the enemy battleentity.`height` is above battleentity.`minheight` + 0.5, the [position](../BattlePosition.md) is set to `Flying`, `Ground` otherwise
- Each enemy party members is checked if it should be included in `avaliabletargets`. In order for the enemy to be included, all of the following must be true (on top of this, if onlyfront is true, only the first one found is included and the rest skipped):
    - Its [position](../BattlePosition.md) isn't `OutOfReach`
    - UndergroundCheck returns true for the attackid and the enemy [position](../BattlePosition.md) which only happens if the `position` isn't `Underground` or the attackid is among the following: -3 (`Moth`'s basic attack), 21 (`FrigidCoffin`), 6 (`BeetleDig`), 27 (`IceDrill`), 18 (`HurricaneBeemerang`)
    - The parameters clauses are fufillfed which is checked using the following rules in order:
        - If onlyground is false, it is included
        - Otherwise, if the [position](../BattlePosition.md) is `Ground`, it is included
        - Otherwise, excludeunderground must be false while [position](../BattlePosition.md) is `Underground`
- `avaliabletargets` is initialised to all the enemy party members included earlier
