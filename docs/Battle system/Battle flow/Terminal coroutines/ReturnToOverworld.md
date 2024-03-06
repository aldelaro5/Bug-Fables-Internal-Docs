# ReturnToOverworld
This terminal coroutine is the last one invoked to transition out of battle. It is always invoked in a finished battle serving as the last step before the battle is completely ended.

```cs
public IEnumerator ReturnToOverworld(bool flee)
```

## Parameters

- `flee`: Whether or not the battle ended as a result of fleeing

## Procedure
The following sections are performed in order.

### Fade out

- We enter a `minipause`
- MainManager.`battlenoexp` is set to whether or not `expreward` is exactly 0
- The `switchicon`, `cancelb` and `hexpcounter` are destroyed
- A fade out to pure black is done via PlayTransition with a speed of 0.075
- A frame is yielded
- All frames are yielded while `transitionobj[0]`'s SpriteRenderer's alpha hasn't reached 95% opacity or MainManager.`musiccoroutine` is still in progress
- 0.25 seconds are yielded
- `transitionobj[0]`'s SpriteRenderer's alpha is set to fully opaque
- A frame is yielded

### Battle teardown

- The `battlemap` is destroyed
- instance.`globalcooldown` is set to 30.0 frames
- A frame is yielded
- MainManager.`map` is enabled
- A frame is yielded
- For each player party members:
    - If the player isn't a `submarine`, their `entity` is enabled
    - All matching `hud` elements have their first child local position set to Vector3.zero
- If the player is a `submarine`, the player is enabled
- If we weren't in a instance.`inevent`:
    - MainManager.ResetCamera is called with instant
    - instance.`camoffset` and instance.`camangleoffset` are reset to their previous counterpart being `oldcamoffset` and `oldcamrotation` respectively
- RestoreLimit is called on the `map` with restoretemp
- A frame is yielded
- The `fronticon` and `expholder` are destroyed
- 0.1 seconds are yielded
- instance.`camspeed` is reset to its previous counterpart being `oldcamspeed`
- A frame is yielded

### Overworld setup

- MainManager.`battlefled` is set to the sent flee
- MainManager.player.entity.`onground` is set to false

The rest has multiple possibilities on how to setup the overworld, but they are mutually exclusive, only the first one mentioned applies. It is possible none applies.

#### Flee is true
This means the battle ended due to the player fleeing:

- MainManager.player.entity.`icooldown` is set to 120.0 frames of invicibility
- If the `caller` exists, some adjustements needs to happen on it (it implies the `caller` is an [Enemy](../../../Entities/NPCControl/Enemy.md#enemy) NPCControl):
    - STOP is called on the `caller` which does the following if the `caller.entity` isn't `dead`:
        - Stop `behaviorroutine` if it was in progress and set it to null
        - Set `forcebehavior` to null and call [StopForceBehavior](../../../Entities/NPCControl/Notable%20methods/StopForceBehavior.md)
        - Call [StopForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#stopforcemove) on the entity
        - Set the entity.`overrideflip` and entity.`overrideonlyflip` to false
        - Set `attacking` to false
    - `caller.entity.overrideanim` is set to false
    - `caller.entity.overrideflip` is set to false
    - `caller.entity.onground` is set to false
    - If the `caller`'s [regionalflag](../../../Flags%20arrays/Regionalflags.md) isn't negative, the slot value is set to false
    - If the `caller`'s [activationflag](../../../Flags%20arrays/flags.md) isn't negative, the slot value is set to false
    - All collisions between the `caller.entity.ccol` and the player.entity.`ccol` are ignored    

#### Flee is false and the `caller` exists
This means the battle ended and it was caused by an [Enemy](../../../Entities/NPCControl/Enemy.md) NPCControl encounter:

- MainManager.player.entity.`icooldown` is set to 120.0 frames of invicibility
- `caller.entity.onground` is set to false
- `callet.entity.rigid` velocity is zeroed out
- The `caller` transform position is set to its `lastpos` (this was set by NPCControl.[StartBattle](../../../Entities/NPCControl/Notable%20methods/StartBattle.md) so the positions is restored to the one before the battle started)
- `caller.attacking` is set to false
- If `expreward` is 0, `enemyfled` is true and instance.`lastdefeated` doesn't contain a `GoldenSeedling` [enemy](../../../Enums%20and%20IDs/Enemies.md):
    - MainManager.`battleenemyfled` is set to true
    - `caller` is disabled
- Otherwise, if the `caller.entity.deathcoroutine` isn't in progress:
    - `caller.entity.deathcoroutine` is set to a new [Death](../../../Entities/EntityControl/Notable%20methods/Death.md) call with activatekill on the `caller`.entity
    - If the `Death0` wasn't playing or it was past the halfway point, it is played at 0.6 volume
    - `caller.entity.spitmoney` is set to `moneyreward`, but it can get multipliers depending on some conditions (these multipliers aren't mutually exclusive, they can stack):
        - `DoublePainReal` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped: 2x
        - [flags](../../../Flags%20arrays/flags.md) 613 is true (RUIGEE is active): 2x
        - `BerryFinder` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped: 1.25x
        - `EXPBoost` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped while [flags](../../../Flags%20arrays/flags.md) 613 is true (RUIGEE is active): 1.25x

#### Flee is false, `enemyfled` is true and `expreward` is 0
The only thing that happens here is MainManager.`battleenemyfled` is set to true

### Other teardown

- A frame is yielded
- instance.`inbattle` is set to false
- [ItemList](../../../ItemList/ItemList.md)'s `listredirect` is set to -1 (this workarounds potential [inlist issues](../../../ItemList/inlist%20issue.md))
- player.entity.`hitwall` is set to false
- CancelAction is called on the player
- [RefreshEntities](../../Actors%20states/RefreshEntities.md) is called with forceanim and refreshmap
- instance.`showmoney` and instance.`hudcooldown` are set to 0.0 (this hides all HUD elements)
- All GameObjects with tag `DelAftBtl` or `Projectile` are destroyed
- The `dimmer` is destroyed
- A frame is yielded

### Fade in
The logic here depends on instance.`inevent`.

#### We weren't in an [event](../../../Enums%20and%20IDs/Events.md)

- If `music[0]` exists:
    - If `overworldmusic` exists and its name isn't the `music[0]`'s name, `music[0]`.volume is set to 0.0
    - If `music[0]`.name is `LevelUp` and `map.nobattlemusic` is true, ChangeMusic is called which will change the music to either silence or to `map.music[map.musicid]` if it exists
- A frame is yielded
- If MainManager.`keepmusicafterbattle` is true, MainManager.`musicresume` is set to `overmusic`
- ChangeMusic is called with the `overworldmusic`. NOTE: It can mean ChangeMusic is called twice, but the second one will stop the first via a StopCoroutine
- MainCamera's parent angles are set to instance.`camangleoffset`
- A frame is yielded
- A fade in from pure black is performed via PlayTransition with a speed of 0.05 which undoes the fade earlier
- We exit the `minipause`

#### We were in an [event](../../../Enums%20and%20IDs/Events.md)

- If `keepmusic` is false, ChangeMusic is called to silence with a fadespeed of 0.075
- ResetEntitySpeed is called which sets all `map.entitities` and all player party members's entity `anim`.speed to 1.0

This implies that events are responsible for doing the fade in themselves as well as the other logic mentioned above in the non event case.

### Final cleanup

- `actiontext` is destroyed
- ForceHitWall is invoked on the player.entity in 0.2 seconds which will set its `hitwall` to false
- Every player party members with an `hp` of 0 has their `hp` set to 1
- CheckAchievement is invoked on the instance in 0.5 seconds
- This BattleControl is destroyed, the battle is now completely over and it cease to exist
