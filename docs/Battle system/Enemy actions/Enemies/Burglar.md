# `Burglar`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true has the following changes:

- The enemy summoning moves that doesn't change the `hp` and `exp` has the usage odds changed when usable to 60% from 40%
- The enemy summoning moves that halves the `hp` and `exp` has the usage odds changed when usable to 45% from 35%
- In the rock throw move, the amount of frames the rock takes to reach its target changes to 35.0 from 50.0

## Move selection
5 moves are possible:

1. A party wide dash attack that may steal an [item](../../../Enums%20and%20IDs/Items.md)
2. A single target rock throw
3. Summons a [Bandit](Bandit.md) or [Thief](Thief.md) enemy without changes to their `hp`, `maxhp` and `exp`
4. Summons a [Bandit](Bandit.md) or [Thief](Thief.md) enemy with half `hp`, `maxhp` and `exp` floored
5. Flees the battle

Move 5 is always used is `holditem` is above -1 meaning this enemy is holding a stolen item. It is the only way to have move 5 used.

Move 1, through 4 have a complex decision process that goes like this:

- If a 4/10 RNG check passes:
    - If this is the last enemy party member left and a 35% RNG check passes (45% instead if hardmode is true)
        - Move 4 is used
    - Otherwise:
        - Move 1 is used
- Otherwise (6/10 odds to fail the RNG check):
    - If this is the last enemy party member left and a 40% RNG check passes (60% instead if hardmode is true)
        - Move 3 is used
    - Otherwise:
        - Move 2 is used

## Move 1 - Dash attack
A party wide dash attack that may steal an [item](../../../Enums%20and%20IDs/Items.md).

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen for each player party member whose `hp` is above 0|This enemy|The player party member|2|null|null|`commandsuccess`|

### Logic sequence

- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) (0.0, 0.0, 0.0) with 2.0 multiplier
- Camera moves to look near (-2.25, 0.0, 0.0)
- Yield all frames until `forcemove` is done
- `Charge10` sound plays
- `Scuttle2` sound plays on loop with 1.05 pitch
- animstate set to 23 (`Chase`)
- Yield for 0.25 seconds
- `Gleam` particles plays with `Gleam` sound at this enemy + (0.0, 1.75 + `height`, -0.1) with 0.5 alivetime
- Yield for 0.5 seconds
- `Scuttle2` sound stops playing
- `trail` set to true
- `FastWoosh` sound plays with 1.1 pitch
- Over the course of 20.0 frames, this enemy moves to (-15.0, 0.0, 0.0) via a lerp. On the first frame that their x position gets lower than -4.0:
    - For each player party members whose `hp` is above 0:
        - DoDamage 1 call happens
        - If `commandsuccess` is false (didn't blocked, ignores FRAMEONE), the player party member has SlowSpinStop with 30.0 in y as spinammount for 60.0 framtime called followed by [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on them
    - If `commandsuccess` is false (didn't blocked, ignores FRAMEONE) and none of the affected player party members had the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition, [StealItem](../StealItem.md) is called for this enemy without always
    - If an item was stolen just now, the local startstate is set to 4 (`ItemGet`)
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `trail` set to false
- position set to (15.0, 0.0, 0.0)
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) startp with 2.0 multiplier using 1 (`Walk`) as walkstate and 0 (`Idle`) as stopstate (however, if an item was stolen just now, 27 (`ItemWalk`) is used as the walkstate and 4 (`ItemGet`) as the stopstate instead)
- Yield all frames until `forcemove` is done

## Move 2 - Rock throw
A single target rock throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on this enemy
- Camera moves to look near the midpoint between this enemy and `playertargetentity`
- A new sprite object is created childed to this enemy using the `MightyPeeble` [medal](../../../Enums%20and%20IDs/Medal.md) sprite positioned at this enemy's `itemoffset`
- animstate set to 4 (`ItemGet`)
- Yield for 0.5 seconds
- Yield all frames until this enemy is `onground`
- animstate set to 28 (`TossItem`)
- Yield for 0.5 seconds
- `Toss` sound plays
- `Fall` sound plays
- The `MightyPeeble` sprite gets rooted
- Over the course of 50.0 frames (35.0 frames instead if hardmode is true), the `MightyPeeble` sprite moves to `playertargetentity` position + (0.0, 1.5, -0.2) via a BeizierCurve with a ymax of 6.0. Before each yield, the sprite Rotate in z by -20.0x the game's frametime
- The `MightyPeeble` sprite gets destroyed
- DoDamage 1 call happens
- Yield for 0.5 seconds

## Move 3 - Enemy summon (no changes to `hp`, `maxhp` and `exp`)
Summons a [Bandit](Bandit.md) or [Thief](Thief.md) enemy party member without changes to the `hp`, `maxhp` and `exp`. No damages are dealt.

### Logic sequence

- Camera moves to look near this enemy
- animstate set to 150
- Yield for 0.25 seconds
- `Whistle` sound plays
- `checkingdead` set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon a [Bandit](Bandit.md) or [Thief](Thief.md) (determined randomly with uniform probabilities) with type `Offscreen` at this enemy + (-2.5, 0.0, -0.2) without cantmove (meaning the enemy will be able to act immediately in the current main turn)
- Yield all frames until `checkingdead` is null (the coroutine completed)

## Move 4 - Enemy summon (half `hp`, `maxhp` and `exp` floored)
Summons a [Bandit](Bandit.md) or [Thief](Thief.md) enemy party member with half `hp`, `maxhp` and `exp` floored. No damages are dealt.

### Logic sequence

- Camera moves to look near this enemy
- animstate set to 150
- Yield for 0.25 seconds
- `Whistle` sound plays
- `checkingdead` set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon a [Bandit](Bandit.md) or [Thief](Thief.md) (determined randomly with uniform probabilities) with type `Offscreen` at this enemy + (-2.5, 0.0, -0.2) without cantmove (meaning the enemy will be able to act immediately in the current main turn)
- Yield all frames until `checkingdead` is null (the coroutine completed)
- The new enemy has their `hp`, `maxhp` and `exp` divided by 2 floored

## Move 5 - Flee
Flees the battle, no damages are dealt.

### Logic sequence

- `tryenemyheal` set to a new [EnemyFlee](../EnemyFlee.md) coroutine with walkstate 27 (`ItemWalk`) with afterimage on this enemy
- Yield all frames until `tryenemyheal` goes to null (the coroutine completed)
- The local fled is set to true indicating a different [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#fled-enemy-post-action) will be done
