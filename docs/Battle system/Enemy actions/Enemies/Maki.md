# `Maki`

## Assumptions
It is assumed that this enemy is fought alongside [Kina](Kina.md) and [Yin](Yin.md). If either or neither are present, the entire enemy action logic will behave in very unexpected ways.

## [StartBattle](../../StartBattle.md) special logic
After loading this enemy, the following special adjustements happens to them if [flags](../../../Flags%20arrays/flags.md) 614 is true (HARDEST is active):

- `maxhp` and `hp` gets increased by 10
- `def` gets incremented

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 2 element with a starting value of 0.

- `data[0]`: A phase tracker that can have 3 possible states depending on the value:
    - 0: The initial state, this enemy hasn't gone through a phase transition. From here, the value is only set to 1 in the pre move logic when [Yin](Yin.md) is no longer present in `enemydata` (they died)
    - 1: This enemy found that [Yin](Yin.md) is no longer present in `enemydata` (they died) in the pre move logic and is ready to perform the phase transition. The value will only be set to 2 when this transition is finished
    - 2: This enemy completed its phase transition in the pre move logic. From there, it is technically still possible to go through the phase transition again with this value, but it's not going to happen in practice because the phase transition also requires this enemy to not have an [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition, but an infinite one was inflicted during the phase transition
- `data[1]`: A shared cooldown on the usage of either the party wide sword slash move or the charging move. When either move is used, `data[1]` is set to 2. When either the single or double sword slash moves is used, the value is decremented after the move's usage. The value needs to reach 0 for the party wide sword slash move or the charging move to become usable again. Effectively, it's a shared antispam of 2 actor turns (not counting the one when the combo attack is used if applicable) for the usage of either moves

## Move selection
5 moves are possible:

1. A single target sword slash
2. A single target double sword slash
3. A party wide aerial sword slash
4. Increment `charge`, then either prepares a big attack or use a move right away that consumes the `charge`
5. A single target double hitting attack involving the consumption of [Kina](Kina.md)'s actor turn that can also hit a nearby player party member as a collateral hit

Move 5 is always (and only used) when `basestate` isn't 13 (`BattleIdle`) meaning move 4 was used previously while [Kina](Kina.md) was present in `enemydata` as this cause `basestate` to change away from 13. 

However, it is still possible for move 5 to not actually be used when `basestate` is 13 if [Kina](Kina.md) became absent from `enemydata` (they died since move 4 was used) or they are currently [stopped](../../Actors%20states/IsStopped.md). When this happens while `basestate` isn't 13, it is set to 13 (alongside the local startstate) and a continue directive is issued to the enemy action loop which restarts the entire action. It effectively means that this enemy will revert to the standard move selection process detailed below which excludes move 5.

As for the other moves, the decision of which move to use is based on odds. However, some moves have additional requirements that must be fufilled for the move to be used if selected. If the move is selected, but their requirements aren't fufilled, either another move will be used or a continue directive will be issued to the enemy action loop which restarts the entire action and effectively rerolls the move selection. The following table contains all these informations for move 1 through 4:

|Move|Odds|Requirements|Falback if requirements aren't fufilled|
|---:|----|----------|------|
|1|4/13|None, the move is always used|N/A|
|2|3/13|[HPPercenkinaHPPercent](../../Actors%20states/HPPercent.md) is 0.6 or lower</li><li>`data[1]` is 0 or lower (the cooldown on this move / the charge move expired)</li></ul>|A continue directive will be issued to the enemy action loop which restarts the entire action and effectively rerolls the move selection|
|4|3/13|<ul><li>`turns` is 2 or above (at least 2 main turns passed since the start of the battle)</li><li>`data[1]` is 0 or lower (the cooldown on this move / the charge move expired)</li></ul>|A continue directive will be issued to the enemy action loop which restarts the entire action and effectively rerolls the move selection|

## Pre move logic
The following logic always happens before the usage of a move if `data[0]` is 0 (the phase transition hasn't happened yet) while [Yin](Yin.md) is no longer present in `enemydata` (they died):

- `Wam` sound plays
- [Emoticon](../../../Entities/EntityControl/EntityControl%20Methods.md#emoticon) called with the `Exclamation` emote for 45 time
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- `data[0]` is set to 1 (mark the phase transition as starting)
- [RemoveCondition](../../Actors%20states/Conditions%20methods/RemoveCondition.md) called to remove the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on this enemy (it's possible [Yin](Yin.md) inflicted it, but it needs to be removed for the phase transition to work)

From there, because `data[0]` is above 0 and this enemy doesn't have the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition, it meets the requirement for the phase transition logic to occur:

- animstate set to 4 (`ItemGet`)
- `Charge7` sound plays
- ShakeSprite called with 0.1 intensity and 45.0 frametimer
- Yield for 0.5 seconds
- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on this enemy for 999999 main turns (effectively infinite) with effect
- Yield for 0.5 seconds
- If `data[0]` is 1 (always the case) and [Kina](Kina.md) isn't present in `enemydata` (they died), they are revived with the following logic:
    - The EntityControl with a `Kina` [animid](../../../Enums%20and%20IDs/AnimIDs.md) is obtained by checking all EntityControl childed to the `battlemap`
    - ShakeSprite called on the `Kina` entity with 0.1 intensity and 30.0 frametimer
    - Yield for 0.5 seconds
    - [AddNewEnemy](../../Actors%20states/Enemy%20party%20members/AddNewEnemy.md) called to add a new [Kina](Kina.md) enemy positioned offscreen at -99.0 in y. NOTE: This bypasses the standard [ReviveEnemy](../ReviveEnemy.md) process which includes a few differences, but the main ones that matters are:
        - `exp` and `money` aren't zeroed out
        - `alreadycounted` is NOT set to true which means that if the player defeats `Kina` again, their defeated counter in their [bestiary entry](../../../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md) will increase by 2 instead of 1
        - [ReorganizeEnemies](../../Actors%20states/Enemy%20party%20members/ReorganizeEnemies.md) with order is NOT called, but in this specific case, it doesn't change anything: `Kina` will still end up at `enemydata[1]` either way
        - Their `hp` won't be restored with a percentage (it gets set to 10 soon after the call instead)
    - Yield for a frame
    - `enemydata[lastaddedid]` position set to the `Kina` entity (the one in `reservedata`)
    - [Heal](../../Actors%20states/Heal.md) called on `enemydata[lastaddedid]` with 10 `hp` (this doesn't actually do anything logically since `Kina` is already at full health here, but it does show the `hp` counter visually)
    - `enemydata[lastaddedid]`'s `hp` is set to 10
    - `enemydata[lastaddedid]` animstate set to 13 (`BattleIdle`)
    - The `Kina` entity found earlier is destroyed. It means only the actual enemy one is left
    - Yield for 0.5 seconds
    - [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
        - text: `commondialogue[179]`
        - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
        - linebreak: `messagebreak`
        - tridimensional: false
        - position: Vector3.zero
        - size: Vector3.one
        - parent: `enemydata[lastaddedid]` (`Kina`)
        - caller: null
    - Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released
    - This enemy (`Maki`) animstate set to 13 (`BattleIdle`)
    - Yield for a frame
    - [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
        - text: `commondialogue[180]`
        - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
        - linebreak: `messagebreak`
        - tridimensional: false
        - position: Vector3.zero
        - size: Vector3.one
        - parent: This enemy (`Maki`)
        - caller: null
    - Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released
- `data[0]` is set to 2 (mark the phase transition as completed)

## Post move logic
The following logic happens after the usage of the single target single and double sword slash moves:

- If `data[1]` is above 0 (the party wide sword slash and charging move cooldowns), it is decremented

## Move 1 - Single sword slash
A single target sword slash.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|8|null|{[NoDamageAnim](../../Damage%20pipeline/DoDamage.md#nodamageanim), [NoSound](../../Damage%20pipeline/DoDamage.md#nosound)}|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between this enemy and `playertargetentity`
- animstate set to 110
- Yield for 0.25 seconds
- `Gleam` particles plays with the `Gleam` sound at this enemy + (0.1, 2.3, -0.1) with 0.5 alivetime
- Yield for 0.5 seconds
- Camera moves to look near `playerdata[playertargetID]`
- `trail` set to true
- `Toss9` sound plays with 1.2 pitch
- [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) to `playertargetentity` position + (2.0, 0.0, -0.1) with 4.0 multiplier with 112 movestate and 113 stopstate
- Yield all frames until `forcemoving` is done
- `HugeHit` sound plays
- `trail` set to false
- DoDamage 1 call happens
- `playertargetentity` animstate set to 11 (`Hurt`), but if `commandsuccess` is true (blocked, ignores FRAMEONE), the animstate is set to 24 (`Block`) instead
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on `playertargetentity` with an h of their `jumpheight` * 2.0
- `playertargetentity`'s y `spin` set to 35.0
- Yield for 0.5 seconds
- animstate set to 13 (`BattleIdle`)
- Yield all frames (up to 30.0 frames) until both this enemy and `playertargetentity` are `onground`
- DeathSmoke particles plays at `playertargetentity` position
- `playertargetentity`'s `spin` zeroed out
- `playertargetentity` animstate set to 18 (`KO`)
- Yield for 0.5 seconds

## Move 2 - Double sword slash
A single target double sword slash.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|5|null|{[NoDamageAnim](../../Damage%20pipeline/DoDamage.md#nodamageanim), [NoSound](../../Damage%20pipeline/DoDamage.md#nosound)}|`commandsuccess`|
|2|Always happen|This enemy|The same `playertargetID` as DoDamage 1|5|null|{[NoDamageAnim](../../Damage%20pipeline/DoDamage.md#nodamageanim), [NoSound](../../Damage%20pipeline/DoDamage.md#nosound)}|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between this enemy and `playertargetentity`
- `Slash2` sound plays
- ShakeSprite called with 0.1 intensity and 45.0 frametimer
- animstate set to 115
- Yield for 0.25 seconds
- Yield for 0.5 seconds
- Camera moves to look near `playerdata[playertargetID]`
- `trail` set to true
- `Toss9` sound plays with 1.2 pitch
- [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) to `playertargetentity` position + (2.0, 0.0, -0.1) with 4.0 multiplier with 112 movestate and 113 stopstate
- Yield all frames until `forcemoving` is done
- `HugeHit` sound plays
- `trail` set to false
- DoDamage 1 call happens
- `playertargetentity` animstate set to 11 (`Hurt`), but if `commandsuccess` is true (blocked, ignores FRAMEONE), the animstate is set to 24 (`Block`) instead
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on `playertargetentity` with an h of their `jumpheight` * 2.0
- `playertargetentity`'s y `spin` set to 35.0
- Yield for 0.5 seconds
- `Toss7` sound plays
- Camera moves a bit up
- Yield all frames until `playertargetentity`'s `rigid` y velocity reaches 0.0 or below (they start to fall)
- [LockRigid](../../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) called on the `playertargetentity` to lock their `rigid` (prevents them to fall further)
- animstate set to 116
- y position increased by 1.0
- SetAnimForce called
- z `spin` set to 20.0
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called with an h of 25.0
- Yield all frames until this enemy's `rigid` y velocity reaches 0.0 or below (they start to fall)
- `spin` zeroed out
- `sprite` angles reset to Vector3.zero
- animstate set to 106
- SetAnimForce called
- x position decreased by 0.5 and y position decreased by 1.0
- Yield all frames until this enemy y position becomes equal or lower than `playertargetentity`'s y position
- Camera moves a bit to the left
- animstate set to 107
- ShakeScreen called with 0.2 ammount and 0.5 time with dontreset
- `HugeHit14` sound plays
- DoDamage 2 call happens
- [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) called on `playertargetentity` to 0.0 in y with 7.0 multiplier using their current animstate as movestate and 18 (`KO`) as stopstate
- Yield all frames (up to 30.0 frames) until both this enemy and `playertargetentity` are `onground`
- ShakeScreen called with 0.3 ammount and 1.0 time with dontreset
- `Thud4` sound plays
- `DirtExplode` particles plays at `playertargetentity` position
- DeathSmoke particles plays at `playertargetentity` position
- `playertargetentity`'s `spin` zeroed out
- `playertargetentity` animstate set to 18 (`KO`)
- Yield for 0.5 seconds

## Move 3 - Party wide aerial sword slash
A party wide aerial sword slash.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|6|null|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence

- `data[1]` is set to 2 (this is the shared cooldown of this move and the charging one)
- Camera moves to look near the midpoint between this enemy and `partymiddle`
- `Charge10` sound plays with 1.1 pitch
- ShakeSprite called with 0.1 intensity and 45.0 frametimer
- animstate set to 110
- Yield for 0.25 seconds
- `Gleam` particles plays with the `Gleam` sound at this enemy + (0.1, 2.3, -0.1) with 0.5 alivetime
- Yield for 0.5 seconds
- Camera moves to look near `partymiddle`
- animstate set to 116
- y position increased by 1.0
- SetAnimForce called
- z `spin` set to 20.0
- Over the course of 51.0 frames, this enemy moves to `partymiddle` + 5.0 in y via a BeizierCurve3 with a ymax of 6.0
- `spin` zeroed out
- `sprite` angles reset to Vector3.zero
- animstate set to 106
- SetAnimForce called
- [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) called to move + 1.0 in y with 20.0 multiplier using 106 as movestate and stopstate
- Yield all frames until `forcemoving` is done
- `trail` set to true
- Camera moves a bit to the left
- [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) called to `partymiddle` with 5.0 multiplier using 106 as movestate and 107 as stopstate
- Yield all frames until `forcemoving` is done
- `trail` set to false
- ShakeScreen called with 0.3 ammount and 1.0 time with dontreset
- `BigHit` sound plays
- `impactsmoke` particles plays at `partymiddle`
- For each player party members:
    - [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on them with an h of 15.0
    - y `spin` set to 20.0
- PartyDamage 1 call happens
- Yield for 0.25 seconds
- Yield all frames (up to 100.0 frames) until `playerdata[0]` (usually `Bee`) is `onground`
- For each player party members:
    - DeathSmoke particles plays at their position
    - `spin` zeroed out on them
- Yield for 0.5 seconds

## Move 4 - Increments `charge`, then prepares a big attack or use a move
Increment `charge`, then either prepares a big attack or use a move right away that consumes the `charge`. No damages are dealt.

### `dontusecharge` set to true
This move sets `dontusecharge` to true if [Kina](Kina.md) is present which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action). If [Kina](Kina.md) is absent, it may be set to false instead which causes no changes.

### Logic sequence

- `data[1]` is set to 2 (this is the shared cooldown with this move and the party wide sword slash one)
- If either [Kina](Kina.md) is present or `charge` is 0 (the latter should always be the case):
    - Camera moves to look near this enemy
    - ShakeSprite called with 0.1 intensity and 45.0 frametimer
    - animstate and `basestate` set to 110 (this slate the usage of the combo attack on the next actor turn, but this will be undone later if [Kina](Kina.md) isn't present meaning they died)
    - `Charge7` sound plays
    - Yield for 0.5 seconds
    - `charge` is incremented
    - [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 4 (green up arrow)
    - `StatUp` sound plays
    - Yield for 0.5 seconds
    - If [Kina](Kina.md) is present, the local startstate is set to `basestate`
- If [Kina](Kina.md) is absent (they died), `basestate` is set to 13 (prevents the combo attack to be used on the next actor turn) and a continue directive is issued to the enemy action loop which restarts the action and effectively uses another move immediately, but it can only be the single or double sword slash move since `data[1]`'s antispam is in effect

## Move 5 - Combo attack with [Kina](Kina.md)
A single target double hitting attack involving the consumption of [Kina](Kina.md)'s actor turn that can also hit a nearby player party member as a collateral hit.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|6|null|{[NoDamageAnim](../../Damage%20pipeline/DoDamage.md#nodamageanim), [NoSound](../../Damage%20pipeline/DoDamage.md#nosound)}|`commandsuccess`|
|2|Always happen|This enemy|The same `playertargetID` as DoDamage 1|5|null|{[NoDamageAnim](../../Damage%20pipeline/DoDamage.md#nodamageanim)}|`commandsuccess`|
|3|Happens if there's at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)) after DoDamage 2|null|The last `playerdata` element whose `hp` is above 0 and is not the target of DoDamage 1 and 2|4|null|null|`commandsuccess`|

### Logic sequence

- The `enemydata` index of [Kina](Kina.md) is obtained
- This enemy `basestate` and the local startstate set to 13 (prevents the usage of this move on the next actor turn)
- [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
    - text: `commondialogue[181]`
    - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - linebreak: `messagebreak`
    - tridimensional: false
    - position: Vector3.zero
    - size: Vector3.one
    - parent: This enemy (`Maki`)
    - caller: null
- Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Yield for a frame
- `Kina` animstate set to 103
- `Toss3` sound plays
- `Slash2` sound plays
- This enemy animstate set to 115
- Yield for 0.5 seconds
- DeathSmoke particles plays at `Kina`'s position with a size of 2.0x
- Yield for a frame
- `Kina` position set to `playertargetentity` position + (4.0, 15.0, -0.1)
- `Kina`'s `trail` set to true
- `Kina` animstate set to 104
- Yield for 0.5 seconds
- Camera moves to look neat `playerdata[playertargetID]`
- This enemy's `trail` set to true
- [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) called on this enemy to move them to `playertargetentity` position + (2.0, 0.0, -0.1) with 4.0 multiplier using 112 as movestate and 113 as stopstate
- Yield all frames until `forcemoving` is done
- This enemy's `trail` set to false
- `Toss9` sound plays with 1.2 pitch
- `HugeHit` sound plays
- Camera moves a bit up right and zooms in
- ShakeScreen called with 0.2 ammount and 0.5 time with dontreset
- DoDamage 1 call happens
- `playertargetentity` animstate set to 11 (`Hurt`)
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on `playertargetentity` with an h of their `jumpheight` * 2.0
- `playertargetentity`'s y `spin` set to 35.0
- Yield all frames until `playertargetentity`'s `rigid` y velocity becomes 0.0 or below (they start falling)
- This enemy animstate set to 13 (`BattleIdle`)
- MainManager.ArcMovement called on this enemy to move them to startp with 10.0 height and 30.0 frametime (done in parallel)
- `Kina`'s `sprite` z angle set to -20.0
- [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) called on `Kina` to move them to `playertargetentity` position + 1.5 in y with 27.0 multiplier using 104 as movestate and 105 as stopstate
- `Toss12` sound plays
- Yield all frames until `forcemoving` is done
- Yield for 0.1 seconds
- DoDamage 2 call happens
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `Kina` animstate set to 101
- `Kina`'s z `spin` set to 30.0
- `Kina`'s `sprite` local y position increased by 1.0
- SetAnimForce called on `Kina`
- `Woosh5` sound plays with 0.8 pitch
- MainManager.ArcMovement called on `Kina` to move them to their position before this action with 3.0 height and 30.0 frametime (done in parallel)
- `playertargetentity`'s `spin` multiplied by -1.0 (it inverts their existing y `spin`)
- If there's at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)), the first one in `playerdata` order whose `hp` is above 0 and it's `playertargetID` will be set to receive the collateral damage
- [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) called to move `playertargetentity` to either the collateral hit receiver position + 1.0 in y if it exists or to their position before this action if it doesn't with 30.0 multiplier using 11 (`Hurt`) as movestate and stopstate
- Yield for 0.5 seconds
- `Drop` sound plays
- `Kina`'s `trail` set to false
- `Kina`'s `spin` zeroed out
- `Kina`'s animstate set to 106
- `Kina`'s `sprite` angles reset to Vector3.zero
- `Kina`'s position set to the value before this action
- SetAnimForce called on `Kina`
- Yield for a frame
- If the collateral receiver exists:
    - DoDamage 3 call happens
    - MainManager.ArcMovement called on `playertargetentity` to move them to their position before this action with 4.0 height and 30.0 frametime (done in parallel)
- DeathSmoke particles plays at `playertargetentity` position
- `playertargetentity` animstate set to 18 (`KO`)
- `playertargetentity`'s `spin` zeroed out
- Yield for 0.5 seconds
- `Kina`'s `cantmove` gets incremented (meaning they will not be able to act this main turn)
