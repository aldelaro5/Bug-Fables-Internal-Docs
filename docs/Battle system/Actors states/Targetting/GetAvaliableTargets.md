# GetAvaliableTargets
This method updates `avaliabletargets` and refreshes the enemy positions when applicable

```cs
private void GetAvaliableTargets(bool onlyground, bool onlyfront, int attackid, bool excludeunderground)
```

## Parameters

- `onlyground`: Only include enemies with a `position` of `Ground` (or `Underground` if excludeunderground is false)
- `onlyfront`: Only the first eligible enemy will be included
- `attackid`: The [skill](../../Enums%20and%20IDs/Skills.md) id or a negative number denoting a member's basic attack (-1 is `Bee`'s, -2 is `Beetle`'s and -3 is `Moth`'s)
- `excludeunderground`: Excludes all enemies with a `position` of `Underground` when `onlyground` is true. There is an overload without this parameter that sends false

## Procedure

There is some dead code where if `currentaction` is `SkillList` and the `AttackArea` of the [skill](../../Enums%20and%20IDs/Skills.md) from `skilldata` is `AllParty` or `SingleAlly`, `availabletargets` is set to `playerdata` which ends the method. This is dead because this method can be called with an attackid being negative which means the method refers to a member's basic attack rather than a skill.

Otherwise:

- RefreshEnemyPos is called which checks all `enemydata` whose `hp` is above 0, whose `cantfall` is false and whose `position` is `Ground` or `Flying`. If the enemy battleentity.`height` is above battleentity.`minheight` + 0.5, the `position` is set to `Flying`, `Ground` otherwise
- Each `enemydata` is checked if it should be included in `avaliabletargets`. In order for the enemy to be included, all of the following must be true (on top of this, if onlyfront is true, only the first one found is included and the rest skipped):
    - Its `position` isn't `OutOfReach`
    - UndergroundCheck returns true for the attackid and the enemy `position` which only happens if the `position` isn't `Underground` or the attackid is among the following: -3 (`Moth`'s basic attack), 21 (`FrigidCoffin`), 6 (`BeetleDig`), 27 (`IceDrill`), 18 (`HurricaneBeemerang`)
    - The parameters clauses are fufillfed which is checked using the following rules in order:
        - If onlyground is false, it is included
        - Otherwise, if the `position` is `Ground`, it is included
        - Otherwise, excludeunderground must be false while `position` is `Underground`
- `avaliabletargets` is initialised to all the `enemydata` included earlier
