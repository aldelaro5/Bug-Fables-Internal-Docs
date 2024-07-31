# `Zommoth`

## Assumptions
It is assumed that if `hologram` is false that the fight is started with `Moth` present in the player party with an [EventStop](../../Actors%20states/BattleCondition/EventStop.md) condition. This assumption doesn't apply if `hologram` is true.

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 2 element with a starting value of 0.

- `data[0]`: This is a tracker to indicate that the first main turns cutscenes with `Moth` completed. A value of 0 allows the multiple cutscenes and scripted parts of the fight to occur. It is only set to 1 at the end of the first usage of the laser attack move where the last cutscene segments happen. The value has no effects and doesn't change if `hologram` is true because in that case, the cutscenes don't apply
- `data[1]`: This is an actor turn cooldown for the usage of the charging move. The value is set to 3 when the move is used and it is decremented if it is above 0 in the post move logic (except when using the laser attack move). The value needs to be 0 or below for the charging move to be usable again. Effectively, it's an antispam of 2 actor turns on the usage of the move not counting actor turns when the laser attack move is used

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the shadow balls projectiles move, the speed of each Projectile calls changes to 27.0 from 35.0
- In the shadow balls projectiles move, the extraargs of each Projectile now ends with `,atkdown@3@2@2` which allows the projectile to potentially inflict [AttackDown](../../Player%20actions/Skills/AttackDown.md)
- In the laser attack move, the amount of frames the laser takes to move each times changes to 51.0 frames from 61.0 frames

## Move selection
7 moves are possible:

1. A single target arm slam attack
2. A single target tail swipe attack
3. A multiple targets shadow balls projectiles throw
4. Summons an enemy from the fungi family
5. Prepares a big attack next actor turn by setting `charge` to 1 and getting the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition
6. A party wide scream attack
7. A party wide laser attack that hits twice with the ability to inflict [AttackDown](../../Player%20actions/Skills/AttackDown.md)

There are 2 move selection process: regular and scripted. The scripted one applies when `hologram` is false and `data[0]` is 0 (haven't yet completed the first main turns cutscenes when they are applicable).

No matter the move selection process, move 7 is always (and only) used when `basestate` isn't 0 (`Idle`) meaning move 5 was used before because this move sets `basestate` away from 0.

### Scripted move selection
If `hologram` is false and `data[0]` is 0 (haven't yet completed the first main turns cutscenes when they are applicable), the move selection process is overriden to the following:

- When `turns` is 0 (first main turn), move 3 is used
- When `turns` isn't 0 (second main turn and later), move 5 is used which sets the `basestate` away from 0 (`Idle`)
- Due to `basestate` not being 0 (`Idle`) move 7 is used on the next actor turn
- `data[0]` is set to 1 as part of move 7 which ends this temporary override

After the override ends or if `hologram` is true, the regular move selection process happens.

### Regular move selection
If `hologram` is true or `data[0]` is 1 (the cutscenes completed or they weren't applicable), the decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|2/15|
|2|3/15|
|3|3/15|
|4|2/15|
|5|2/15|
|6|3/15|

However, if move 4 is selected and this enemy is the last remaining `enemydata` left, a continue directive is issued to the enemy action loop which restarts the entire action and effectively rerolls the move selection process.

This also happens if move 5 is selected while `data[1]` is above 0 meaning the cooldown on the usage of that move hasn't yet expired.

## Post move logic
The following logic is always done after using a move except for the laser attack move:

- If `data[1]` is above 0, it is decremented

## Move 1 - Arm slam attack
A single target arm slam attack.

### [BasicAttack](../../Damage%20pipeline/BasicAttack.md) calls

|#|Conditions|enemyid|walkstate|attackstate|attackstate2|damage|offset|property|shake|delay|sounds|dontgettarget|
|-:|---------|-------|--------|------------|------------|------|-----|--------|-----|-----|------|-------------|
|1|Always happen|This enemy|1 (`Walk`)|100|101|3|(1.45, 0.0, -0.1)|null|0.0|Random between 0.5 and 0.75 seconds|`ZammothMelee1,ZammothMelee2`|false|

### Logic sequence

- `checkingdead` is set to the BasicAttack 1 call
- Yield all frames until `checkingdead` is null (the BasicAttack 1 call completed)

## Move 2 - Tail swipe attack
A single target tail swipe attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|4|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (2.5, 0.0, -0.1) with 2.0 multiplier using 1 (`Walk`) as walkstate and 102 as stopstate
- Yield all frames until `forcemove` is done
- `ZammothMelee1` sound plays
- ShakeSprite called with 0.1 intensity and 30.0 frametimer
- Over the course of 31.0 frames, `model`'s y angle changes to -30.0 via LerpAngle
- `ZammothSwipe` sound plays
- `model` angles reset to Vector3.zero
- animstate set to 103
- MainManager.spin called in parallel on this enemy towards y angles of 360.0 for 30.0 frametime with smooth
- Yield for 0.25 seconds
- DoDamage 1 called
- If `commandsuccess` is false (didn't blocked, ignores FRAMEONE), `enemybounce` is set to a new array of one element being a BounceEnemy call on `playerdata[playertargetID]` using routineid 0 for 1 bounce and 1.25 multiplier
- Yield for 0.5 seconds
- Yield all frames until `enemybounce[0]` is done if it was started

## Move 3 - Shadow balls projectiles
A multiple targets shadow balls projectiles throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen 2 times, but the second call requires that at least 1 player party member is alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|3|[Poison](../../Damage%20pipeline/AttackProperty.md)|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) (target changes for each calls)|A new `Prefabs/Objects/ShadowBall` GameObject rooted positioned at this enemy position + (1.5, 3.0, -0.1) on the first hit and this enemy position + (-1.35, 3.0, -0.1) on the second hit|35.0 (27.0 instead if hardmode is true)|0.0|`SepPart@3@1,keepcolor` (`SepPart@3@1,keepcolor,atkdown@3@2@2` instead if hardmode is true)|`Stars`|`PingShot`|null|Vector3.zero|false|

### Logic sequence

- Camera moved to look near the midpoint between this enemy and `partymiddle`
- `ZammothMelee1` sound plays
- animstate set to 102
- ShakeSprite called with 0.1 intensity and 30.0 frametimer
- Yield for 0.5 seconds
- ShakeScreen called with 0.05 ammount and 0.5 time
- `ZammothOrb1` sound plays
- 2 projectiles elements are initialised:
    - Each is a new `Prefabs/Objects/ShadowBall` GameObject rooted positioned at this enemy position + (0.0, 3.0, -0.1)
    - MainManager.MoveTowards called on the object to move it in x by +1.5 on the first one and -1.35 on the seocond one for 10.0 frametime with smooth without local (started in paralel)
- animstate set to 104
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Yield for 0.5 seconds
- animstate set to 105
- Yield for 0.33 seconds
- Done 2 times (each hit):
    - `ZammothOrb2` sound plays
    - Projectile 1 call happens
    - If this is the first hit:
        - Yield for 0.33 seconds
        - Yield for 0.33 seconds
        - If there's not at least 1 player party member is alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)), the second projectile element is destroyed and the second hit doesn't happen
        - Otherwise (the second hit will happen), [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) is called
- Yield all frames until both projectiles objects are null (all Projectile 1 calls completed)
- Yield for 0.5 seconds

## Move 4 - Enemy summon
Summons an enemy from the fungi family. No damages are dealt

### Logic sequence

- Camera moves to look near this enemy
- `ZammothRumble` sound plays
- animstate set to 102
- ShakeSprite called with 0.1 intensity and 30.0 frametimer
- Yield for 0.5 seconds
- `ZammothSummon` sound plays
- ShakeScreen called with 0.075 ammount and 0.5 time
- animstate set to 104
- `checkingdead` set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon an enemy from the fungi family with type `Offscreen` at (1.3, 0.0, 0.45). The enemy to summon is determined randomly among the following with uniform odds:
    - [Zombee](Zombee.md)
    - [CordycepsAnt](CordycepsAnt.md)
    - [Zombeetle](Zombeetle.md)
    - [Mushroom](Mushroom.md)
    - [Bloatshroom](Bloatshroom.md)
- Yield all frames until `checkingdead` is null (the coroutine completed)
- `enemydata[lastaddedid]`'s `exp` gets divided by 2 floored

## Move 5 - Prepares a big attack with 1 `charge` and [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md)
Prepares a big attack next actor turn by setting `charge` to 1 and getting the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition. No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- Camera moves to look near this enemy
- animstate set to 102
- `ZammothRumble` sound plays
- `ZammothOrb1` sound plays with 0.4 pitch
- ShakeScreen called with 0.075 ammount and 0.5 time
- ShakeSprite called with 0.1 intensity and 60.0 frametimer
- Done in a separate coroutine started in paralel called MainManager.Waves:
    - A `Prefabs/Objects/SphereGlowEffect` GameObject is instantiated rooted at this enemy's `model`'s first child's first child (purple particles in their chest) with a scale of (50.0, 50.0, 1.0) with a pure magenta color
    - Over the course of 81.0 frames, the scale of `Prefabs/Objects/SphereGlowEffect` changes to 0.0x via a lerp
    - The `Prefabs/Objects/SphereGlowEffect` gets destroyed
- Yield for 1 second
- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition to this enemy for 2 main turns (effectively 1 as the current main turn is advanced soon) with effect
- `charge` set to 1
- `basestate` and the local startstate set to animstate (this allows to use the laser attack move on the next actor turn)
- Yield for 0.5 seconds
- If `hologram` is false and `data[0]` is 0 (haven't yet completed the first main turns cutscenes when they are applicable):
    - [SetDefault](../../Visual%20rendering/SetDefaultCamera.md) called
    - Yield for 0.5 seconds
    - If `Moth` from the player party has an `hp` of 0 or below, [RevivePlayer](../../Actors%20states/Player%20party%20members/RevivePlayer.md) is called on them leaving them with 1 `hp` without showcounter
    - [SetText](../../../SetText/SetText.md) is called in [dialogue](../../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
        - text: `|`[tail](../../../SetText/Individual%20commands/Tail.md)`,0|` followed by `map.dialogues[5]`
        - [fonttype](../../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
        - linebreak: `messagebreak`
        - tridimensional: false
        - position: Vector3.zero
        - size: Vector3.one
        - parent: null
        - caller: null
    - Yield all frames until the [message](../../../SetText/Notable%20states.md#message) lock is released
- `data[1]` is set to 3 (the move won't be usable for the next 3 actor turns including the laser attack one)

## Move 6 - Scream attack
A party wide scream attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|3|[DefDownOnBlock](../../Damage%20pipeline/AttackProperty.md)|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence

- `checkingdead` set to a new ScreamWaves call (`checkingdead` is set to null when completed)
- Yield all frames until `checkingdead` is null

This is what the coroutine ends up doing:

- Camera moves to look near this enemy
- animstate set to 102
- ShakeSprite called with an intensity of (0.05, 0.0, 0.0) and 85.0 frametimer
- Yield for 0.85 seconds
- `Gleam` particles plays with the `Gleam` sound at this enemy's `sprite` + its [CenterPos](../../Actors%20states/CenterPos.md) + (0.5, 1.0, -0.1) with 0.5 alivetime
- Yield for 0.5 seconds
- Done 5 times in a separate coroutine started in paralel called MainManager.Waves:
    - A `Prefabs/Objects/SphereGlowEffect` GameObject is instantiated rooted at this enemy's world [CenterPos](../../Actors%20states/CenterPos.md) with a scale of 0.0x
    - Over the course of 31.0 frames, the scale of `Prefabs/Objects/SphereGlowEffect` changes to (50.0, 50.0, 1.0) via a lerp
    - The `Prefabs/Objects/SphereGlowEffect` gets destroyed
    - Yield for 0.25 seconds
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `ZammothSummon` sound plays
- animstate set to 104
- ShakeScreen called with an ammount of 0.2 and 1.0 time
- Yield for 0.25 seconds
- PartyDamage 1 call happens
- Yield for a second
- `checkingdead` set to null which signals the caller that this coroutine completed

## Move 7 - Laser attack
A party wide laser attack that hits twice with the ability to inflict [AttackDown](../../Player%20actions/Skills/AttackDown.md).

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|3|[Poison](../../Damage%20pipeline/AttackProperty.md)|`commandsuccess`|0.0|Vector3.zero|false|null|
|2|Always happen|This enemy|2|[Poison](../../Damage%20pipeline/AttackProperty.md)|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence

- ShakeScreen called with 0.075 ammount and 0.5 time with dontreset
- `ZammothRumble` sound plays
- ShakeSprite called with 0.1 intensity and 30.0 frametimer
- Camera moves to look near this enemy
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- `ZammothBeam` sound plays with 0.8 pitch
- ShakeScreen called with 0.075 ammount and -1.0 time with dontreset
- Camera moves to look near the midpoint between this enemy and `partymiddle`
- animstate set to 103
- Yield for a frame
- If `hologram` is false and `data[0]` is 0 (haven't yet completed the first main turns cutscenes when they are applicable):
    - If `Moth` from the player party has an `hp` of 0 or below, [RevivePlayer](../../Actors%20states/Player%20party%20members/RevivePlayer.md) is called on them leaving them with 1 `hp` without showcounter
    - [RemoveCondition](../../Actors%20states/Conditions%20methods/RemoveCondition.md) is called to remove the [EventStop](../../Actors%20states/BattleCondition/EventStop.md) condition from `Moth`
    - `Moth`'s animstate set to 116
    - Yield for 0.5 seconds
    - `Shield` sound plays
    - For each player party members:
        - [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition on them for 1 main turn (but the condition will be manually removed later in this move)
        - Their `shieldenabled` is set to true
    - Yield for 0.5 seconds
- `BigLazer` particles plays at this enemy's `model`'s first child's first child position (purple particles in their chest) with angles of (-40.0, -90.0, -90.0) and -1.0 alivetime
- Yield for 0.5 seconds
- Over the course of 61.0 frames (51.0 frames instead if hardmode is true), `BigLazer` particles's x angle changes to 0.0 via a LerpVectorAngle. On the first frame after 42.0 frames (35.0 frames instead if hardmode is true):
    - PartyDamage 1 call happens
    - All player party members whose `hp` is above 0 and without the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition has [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called on them to inflict them the [AttackDown](../../Player%20actions/Skills/AttackDown.md) condition for 3 main turns (effectively 2 since the current main turn advanced soon) with effects
- Yield for 0.5 seconds
- Over the course of 61.0 frames (51.0 frames instead if hardmode is true), `BigLazer` particles's x angle changes to 40.0 via a LerpVectorAngle. On the first frame after 18.0 frames (15.0 frames instead if hardmode is true):
    - PartyDamage 2 call happens
    - All player party members whose `hp` is above 0 and without the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition has [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called on them to inflict them the [AttackDown](../../Player%20actions/Skills/AttackDown.md) condition for 3 main turns (effectively 2 since the current main turn advanced soon) with effects
- Yield for 0.5 seconds
- GradualScale called to change the scale of `BigLazer` particles to (0.0, 0.0, 1.0) over the course of 31.0 frames then destroy the object
- Yield for 0.5 seconds
- MainManager.`screenshake` reset to Vector3.zero
- `basestate`, animstate and the local startstate all set to 0 (`Idle`) which prevents to use this move again on the next actor turn
- If `hologram` is false and `data[0]` is 0 (haven't yet completed the first main turns cutscenes when they are applicable):
    - `Moth`'s `cantmove` is set to 1 (meaning they need to have their actor turn advanced once before they can act)
    - `Moth`'s animstate set to 13 (`BattleIdle`)
    - For each player party members:
        - [RemoveCondition](../../Actors%20states/Conditions%20methods/RemoveCondition.md) called to remove their [Shield](../../Actors%20states/BattleCondition/Shield.md) condition
        - Their `shieldenabled` is set to false
    - [SetText](../../../SetText/SetText.md) is called in [dialogue](../../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
        - text: `|`[tail](../../../SetText/Individual%20commands/Tail.md)`,0|` followed by `map.dialogues[6]`
        - [fonttype](../../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
        - linebreak: `messagebreak`
        - tridimensional: false
        - position: Vector3.zero
        - size: Vector3.one
        - parent: null
        - caller: null
    - Yield all frames until the [message](../../../SetText/Notable%20states.md#message) lock is released
    - `data[0]` is set to 1 indicating that all of the first main turns cutscenes completed
