# [UltimaxTank](../../Enemy%20actions/Enemies/UltimaxTank.md) EventDialogue
`UltimaxTank` has an EventDialogue that is meant to trigger as their [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath). This is used a transition to a different `enemydata` set containing only a [WaspGeneral](../../Enemy%20actions/Enemies/WaspGeneral.md).

## Logic sequence

- The `enemydata` index of `UltimaxTank` is obtained
- `UltimaxTank`'s `eventondeath` is set to -1 which disables this EventDialogue so [CheckDead](../Action%20coroutines/CheckDead.md) won't run it again
- [AddNewEnemy](../../Actors%20states/Enemy%20party%20members/AddNewEnemy.md) called to add a [WaspGeneral](../../Enemy%20actions/Enemies/WaspGeneral.md) enemy at (`UltimaxTank`'s x position, 20.0, 0.0)
- Yield for a frame
- All enemy party members has their animstate set to 11 (`Hurt`)
- `UltimaxTankShakingDeathLoop` sound plays on loop using `sounds[9]` with 0.75 volume
- ShakeScreen called with an ammounnt of 0.1 and -1.0 time (infinite)
- [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
    - text: `commondialogue[171]`
    - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - linebreak: `messagebreak`
    - tridimensional: false
    - position: Vector3.zero
    - size: Vector3.one
    - parent: `UltimaxTank`
    - caller: null
- Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released. On the first frame every 30.0 frames, DeathSmoke particles plays at `UltimaxTank` position + (A random number between -1.0 and 3.0, 2.0, -2.0)
- ShakeScreen called with 0.2 ammount and 3.0 time
- Yield for a second
- `UltimaxTankShakingDeathLoop` sound stopped
- `UltimaxTankDeathExplosion` sound plays
- `sounds[9]` stopped
- `explosion` particles plays at `UltimaxTank` position with a scale of 5.0x
- Every enemy party member other than `UltimaxTank` and `WaspGeneral` has MainManager.ArcMovement called on them to move them to a random vector between (-10.0, 3.0, -2.0) and (10.0, 3.0, 2.0) with 10.0 height and 29.0 frametime (each are done in paralel)
- Yield for 0.25 seconds
- `UltimaxTank` moved offscreen at -999.0 in y
- Yield for a frame
- Yield for 0.25 seconds
- Every enemy party members except `WaspGeneral` has [CleanKill](../../Actors%20states/CleanKill.md) called on them to turn them into empty `enemydata` shells with a startp of Vector3.zero
- [ReorganizeEnemies](../../Actors%20states/Enemy%20party%20members/ReorganizeEnemies.md) called (this will remove `UltimaxTank` and the enemies they summoned if any from the enemy party completely as they just died)
- Yield for a second
- Yield for a second
- `WaspGeneral` animstate set to 11 (`Hurt`)
- y `spin` set to 20.0
- `WaspGeneral` [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) to 0.0 in y and z with 30.0 frametime without changeanim
- `Fall` sound plays with 0.8 pitch
- Yield for a frame
- Yield all frames until `forcemoving` is done
- `Death3` sound plays
- ShakeScreen called with an ammount of 0.1 with 0.5 time
- `WaspGeneral` animstate set to 102
- `WaspGeneral`'s `spin` zeroed out
- DeathSmoke particles plays at `WaspGeneral` position
- Yield for 0.5 seconds
- ShakeSprite called on `WaspGeneral` with 0.1 intensity and 30.0 frametimer
- Yield for 0.5 seconds
- `Jump` sound plays
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on `WaspGeneral`
- `WaspGeneral`'s animstate and `basestate` set to 13 (`BattleIdle`)
- Yield for a second
- `sounds[9]` stopped
- MainManager.`screenshake` zeroed out
- `checkingdead` set to a new [CheckDead](../Action%20coroutines/CheckDead.md) call (this won't do anything because only `WaspGeneral` exists and they are alive)
- Yield all frames until `checkingdead` is null (the CheckDead completed)