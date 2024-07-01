# [Acolyte](../../Enemy%20actions/Enemies/Acolyte.md) and `AcolyteVine` EventDialogue
These 2 enemies have a special interactions where it is possible for an `Acolyte` to spawn an `AcolyteVine` and get on top of it which changes their [position](../../Actors%20states/BattlePosition.md) to `Flying`. Dropping down to `Ground` from there can only happen in 2 ways:

- `Acolyte` gets an actor turn where they attack and then drop to `Ground` also killing their `AcolyteVine`
- `Acolyte` or `AcolyteVine` dies

`Acolyte` is assumed to have [cantfall](../../Actors%20states/Enemy%20features.md#cantfall) set to true meaning it's not possible for the player to make them drop down without them doing their aerial strike unless `Acolyte` or `AcolyteVine` dies. The problem is this isn't something that the damage pipeline can accomdate and there's also the issue that if either dies, `Acolyte` needs to drop down first and `AcolyteVine` needs to die before `Acolyte` gets killed if they were defeated.

This is why an EventDialogue for either enemies exists on their [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath): it allows the game to ensure that this drop down logic occurs as well as making sure that `AcolyteVine` dies no matter if they died or `Acolyte` did. It also handles other custom logic as part of the drop down that the damage pipeline cannot handle.

## About status resistance
If `Acolyte`'s `hp` is above 0 when this EventDialogue triggers, the dropdown logic will undo any resitance increases of [Poison](../../Actors%20states/BattleCondition/Poison.md), [Freeze](../../Actors%20states/BattleCondition/Freeze.md), [Numb](../../Actors%20states/BattleCondition/Numb.md) and [Sleep](../../Actors%20states/BattleCondition/Sleep.md) conditions.

## Logic sequence

If there's more than 1 enemy party members:

- `Acolyte` and `AcolyteVine`'s `enemydata` indexes are obtained if they exists
- If `AcolyteVine` is present:
    - `AcolyteVine` animstate set to 18 (`KO`)
    - `AcolyteVine`'s y `spin` set to 20.0
    - `AcolyteVine`'s `rotater` gets a DialogueAnim added that will cause them to shrink to Vector3.zero
    - `Vine2` sound plays
    - If `Acolyte`'s `hp` is above 0, their animstate is set to 11 (`Hurt`)
    - ShakeScreen called with an ammount of (0.05, 0.05, 0.05) and 0.5 time
    - Yield for 0.6 seconds
    - If `Acolyte`'s `hp` is above 0, `Fall` sound plays
    - `ChargeDown` sound plays
    - Over the course of 20.0 frames:
        - If `Acolyte`'s `hp` is above 0, `Acolyte`'s `height` lerped from 3.4 to 0.2. Otherwise, `ItemBounce0` plays on the first frame only
        - `AcolyteVine`'s `startscale` changes towards Vector3.zero
        - `AcolyteVine`'s position increases by framestep * -0.2 each frame
    - `AcolyteVine` moved offscreen at 9999.0 in y
    - If `Acolyte`'s `hp` is above 0:
        - [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on `Acolyte`
        - Yield for 0.5 seconds
        - Yield all frames until `Acolyte` is `onground`
        - Yield for 0.2 seconds
        - `Acolyte`'s [position](../../Actors%20states/BattlePosition.md) changed to `Ground`
        - `AcolyteVine` is destroyed
        - `Acolyte`'s `height` set to 0.0
        - `Acolyte`'s `sprite` local position reset to Vector3.zero
        - `Acolyte`'s `basestate` set to 13 (`BattleIdle`)
        - `Acolyte`'s `cursoroffset`, `poisonres`, `freezeres`, `numbres` and `sleepres` are reset to their default value from [GetEnemyData](../../../TextAsset%20Data/Enemies%20data.md#getenemydata) without createentity
- If `Acolyte` is still present, but their `hp` is 0:
    - `Acolyte`'s `eventondeath` set to -1 (so this EventDialogue isn't recursively called)
    - A [CheckDead](../Action%20coroutines/CheckDead.md) is done which will cleanly kill `Acolyte`

Otherwise (if the enemy killed just now is the only one left which is normally `Acolyte`):

- The enemy's `eventondeath` set to -1 (so this EventDialogue isn't recursively called)
- A [CheckDead](../Action%20coroutines/CheckDead.md) is done which will cleanly kill the enemy

In either case, [ReorganizeEnemies](../../Actors%20states/Enemy%20party%20members/ReorganizeEnemies.md) is called
