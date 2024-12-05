# UpdateAnim
This method updates the battle entities's [animstate](../../Entities/EntityControl/Animations/animstate.md) considering various conditions.

```cs
private void UpdateAnim(bool onlyplayer)
```

## Parameters

- `onlyplayer`: Tells if only the player party members should be updated. There is an overload without this parameter that sends false

## Procedure
There are 2 parts to this: the player party members one and the enemy party members one. The latter is only done if onlyplayer is false.

### Players
This applies to each element of `playerdata`.

The enablement of the corresponding `tiredpart` has its enablement changed to be enabled only when the player party member's `cantmove` is 0 or below (they have at least 1 actor turns available), the player `hp` is above 0 and its `tired` is above 0. It is disabled otherwise.

The rest of the logic isn't done if the player's battleentity.`overrideanim` is true.

- If the `hp` is 0 or below, battleentity.`animstate` is set to 18 (`KO`)
- Otherwise, if the player has the [EventStop](../Actors%20states/BattleCondition/EventStop.md), [Freeze](../Actors%20states/BattleCondition/Freeze.md) or [Numb](../Actors%20states/BattleCondition/Numb.md) condition, battleentity.`animstate` is set to 11 (`Hurt`)
- Otherwise, if the `currentturn` is the player index (it's the one being selected for taking action) and [currentaction](../Player%20UI/Pick.md) is `BaseAction` (the main vine action menu):
    - If the `hp` is above 4 and the player doesn't have the [Poison](../Actors%20states/BattleCondition/Poison.md) or [Fire](../Actors%20states/BattleCondition/Fire.md) condition, battleentity.`animstate` is set to 19 (`PickAction`)
    - Otherwise, battleentity.`animstate` is set to 20 (`WeakPickAction`)
- Otherwise, battleentity.`animstate` is only changed if `blockcooldown` expired:
    - If the player has the [Sturdy](../Actors%20states/BattleCondition/Sturdy.md) condition, battleentity.`animstate` is set to 24 (`Block`)
    - If the player has the [Sleep](../Actors%20states/BattleCondition/Sleep.md) condition, battleentity.`animstate` is set to 14 (`Sleep`)
    - If the `hp` is above 4 and the player doesn't have the [Poison](../Actors%20states/BattleCondition/Poison.md) or [Fire](../Actors%20states/BattleCondition/Fire.md) condition (it also implies it's not the currently selected player as it was ruled out above):
        - If the player has the [Taunted](../Actors%20states/BattleCondition/Taunted.md) condition or it has the `Berserker` [medal](../../Enums%20and%20IDs/Medal.md) equipped, battleentity.`animstate` is set to the return of AngryAnim which is a value that depends on the player [animid](../../Enums%20and%20IDs/AnimIDs.md): 10 (`Flustered`) for `Bee`, 5 (`Angry`) for `Beetle` and 102 for `Moth`
        - Otherwise, battleentity.`animstate` is set to 13 (`BattleIdle`)
    - Otherwise, battleentity.`animstate` is set to 17 (`WeakBattleIdle`)

### Enemies
This applies to each element of `enemydata` if onlyplayer is false.

battleentity.`animstate` is only set if the `hp` is above 0 and the entity.`droproutine` is not in progress.

- If the enemy has the [Flipped](../Actors%20states/BattleCondition/Flipped.md) condition:
    - If the enemy also has the [Sleep](../Actors%20states/BattleCondition/Sleep.md) condition, battleentity.`animstate` is set to 25 (`SleepFallen`)
    - If it doesn't have it, battleentity.`animstate` is set to 15 (`Fallen`)
- Otherwise, if the enemy has the [Freeze](../Actors%20states/BattleCondition/Freeze.md) or [Numb](../Actors%20states/BattleCondition/Numb.md) condition, battleentity.`animstate` is set to 11 (`Hurt`)
- Otherwise, if the enemy has the [Sleep](../Actors%20states/BattleCondition/Sleep.md) condition, battleentity.`animstate` is set to 14 (`Sleep`)
- Otherwise, if the enemy [holditem](../Actors%20states/Enemy%20features.md#holditem-and-helditem) isn't -1 (the enemy is holding an item), battleentity.`animstate` is set to 4 (`ItemGet`)
- Otherwise, if the enemy [isdefending](../Actors%20states/Enemy%20features.md#defenseonhit-and-isdefending), battleentity.`animstate` is set to 24 (`Block`)
- Otherwise, battleentity.`animstate` is set to battleentity.`basestate`
