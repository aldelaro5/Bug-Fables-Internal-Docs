# `FlyTrap`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) or [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) infliction move on a [MotherChomper](MotherChomper.md) has its RNG check odds changed to 41% from 34%
- The biting attack move has its RNG check odds changed to 65% from 75%
- Due to the biting attack move change, the enemy spawn move usage odds changes to 35% from 25%, but the RNG check for whether to actually spawn the enemy also changes to 7.5/10.0 from 5.0/10.0 bringing the overall odds to 26.25% from 12.5% to spawn a new enemy
- If an attempt was made to spawn a new enemy and either the RNG check failed or no suitable location could be found, this enemy will do a biting attack instead of ending their actor turns immediately after

## Move selection
3 moves are possible as well as the possibility of doing none of them:

1. Inflicts [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) or [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) on a [MotherChomper](MotherChomper.md) enemy party member
2. A single target bite attack that may drain `hp`
3. Summon another `FlyTrap` enemy

All moves don't just have base odds: they all are selected based on multiple conditions.

Move 1 can only be used and will always be used if all of the following conditions are fufilled:

- A [MotherChomper](MotherChomper.md) enemy party member is present (the first one will be selected)
- A 34% RNG check passes (41% instead if hardmode is true)
- The found [MotherChomper](MotherChomper.md) enemy party member doesn't already have the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) or [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) conditions

As for move 2, it will be used if any of the following conditions are fufilled:

- A failed attempt at using move 3 just occured on the same actor turn and hardmode is true
- There are 4 enemy party members (a full enemy party)
- `locktri` is true (meaning this is an enemy that was spawned from another `FlyTrap`)
- A 75% RNG check passes (65% instead if hardmode is true)

If neither move 1 or 2 are used, move 3 will be attempted to be used. It is however possible that the attempt to use the move fails if any of the following conditions occurs:

- No suitable location was found to spawn the new enemy after trying to generate one 20 times in a row
- An RNG check fails and it has 5.0/10.0 chance to pass (7.5/10.0 instead if hardmode is true)

If neither of the fail conditions happens, move 3 will be used. If any happens however, what happens next depends if hardmode is true or not. If it's true, move 2 will be used instead, but if it's false, the actor turn ends meaning it is possible the enemy doesn't end up using any moves for their actor turn.

## Move 1 - Boost on a [MotherChomper](MotherChomper.md)
Inflicts [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) or [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) on a [MotherChomper](MotherChomper.md) enemy party member. No damages are dealt.

### Logic sequence

- The first [MotherChomper](MotherChomper.md) enemy party member index is obtained
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) near `MotherChomper` at 2.0 multiplier with 23 (`Chase`) walkstate and 0 (`Idle`) stopstate
- Yield all frames until `forcemove` is done
- Done 3 times:
    - `flip` set to true except on the last iteration where it's set to false
    - [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
    - `Jump` sound plays
    - Yield for 0.3 seconds
    - Yield all frames until `onground` is true
- `Spin8` sound plays
- `flip` set to false
- y `spin` set to 25.0
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called with an h of `jumpheight` * 1.5
- For the next 60.0 frames, `spin` is zeroed out whenever 15.0 frames passed and `onground` is true
- `StatUp` sound plays
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) or [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) on the `MotherChomper` enemy party member for 2 main turns. The odds to inflict `AttackUp` are 6/10 while it's 4/10 to inflict `DefenseUp`
- Yield for 0.6 seconds

## Move 2 - Bite attack
A single target bite attack that may drain `hp`

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|2 (1 instead if `locktri` is true meaning this enemy was spawned from another `FlyTrap`)|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playertargetentity`
- `Scuttle3` sound plays on this enemy on loop
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (0.85, 0.0, -0.1) with a 2.5 multiplier using 23 (`Chase`) as walkstate and 100 as stopstate
- Yield all frames until `forcemove` is done
- `Sucttle3` sound stops looping
- `Chew` sound plays
- Yield for 0.33 seconds
- animstate set to 102
- `Bite` sound plays
- Yield for 0.05 seconds
- DoDamage 1 call happens
- If `commandsuccess` is false (didn't blocked, ignores FRAMEONE):
    - `hp` is increased by `lastdamage` * 0.75 ceiled then clamped from 0 to `maxhp`
    - [ShowDamageCounter](../../Visual%20rendering/ShowDamageCounter.md) called with type 1 (HP counter) starting at this enemy + Vector3.up and ending at this enemy + 3.0 in y
    - `Heal` sound plays
- Yield for 0.5 seconds
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) startp at 2.5 multiplier with 23 (`Chase`) walkstate and 0 (`Idle`) stopstate
- Yield all frames until `forcemove` is done

## Move 3 - `FlyTrap` summon
Summon another `FlyTrap` enemy. No damages are dealt.

It is possible that attempting this move fails which results in the actor turn ending when hardmode is false or move 2 (bite attack) being used instead when hardmode is true.

### Logic sequence

- Done 3 times:
    - `flip` is toggled
    - `FlipNoise3` sound plays
    - [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
    - Yield for 0.15 seconds
    - Yield all frames until `onground` is true
    - Yield a frame
- `Spin8` sound plays
- `flip` set to false
- y `spin` set to 25.0
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called with an h of `jumpheight` * 1.5
- A new sprite object is created rooted at this enemy position + 2.0 in y using the `ChomperSeed` [item](../../../Enums%20and%20IDs/Items.md) sprite on the same layer than this enemy
- A Vector3 is attempted to be generated that will corresponds to the position to spawn the new enemy. Each attempt may fail, but up to 20 attempts will be done and the first one that fufills some conditions will be used. Each attempts generates a vector where the x component is between 0.25 and 7.6, the y component is 0.0 and the z component is between -0.75 and 0.75. For the location to be accepted, any enemy part member needs to be located at a distance less than its `size` * 1.8. If no suitable locations is found after 20 attempts, the logic will proceed assuming the last one generated (but the spawning won't happen later)
- Over the course of 60.0 frames, the `ChomperSeed` sprite moves to the generated position via a BeizierCurve3 with a ymax of 6.0. Before each frame yield, the z angles increases by 20x the game's frametime. If it's been more than 15.0 frames while `onground` is true, `spin` is zeroed out
- `spin` is zeroed out again
- DeathSmoke particles plays at the `ChomperSeed` sprite
- Yield a frame
- The `ChomperSeed` sprite is destroyed
- `Woosh3` sound plays
- If the Vector3 was accepted earlier and a 5.0/10.0 RNG test passes (7.5/10.0 instead if hardmode is true), the spawning will occur:
    - [AddNewEnemy](../../Actors%20states/Enemy%20party%20members/AddNewEnemy.md) called to add another `FlyTrap` enemy at the generated location with the `SeedMini` animation
    - Yield a frame
    - Yield all frames until `checkingdead` is null (the enemy summon is complete)
    - The new `FlyTrap` has several fields adjustements:
        - `cantmove`: 1 (can only act starting next main turn)
        - `cursoroffset`: increased by (0.1, -0.5, 0.0)
        - `hp`: The `hp` of the last `enemydata` * 0.75 floored
        - `maxhp`: The `maxhp` of the last `enemydata` * 0.75 floored
        - `locktri`: true (prevents this enemy from using this move so recursive summons aren't possible)
        - `def`: 0 (this cancels the effects of HARDEST being active and applicable)
        - `exp`: halved floored
        - `money`: halved floored
    - Yield for 0.5 seconds
- Otherwise (the spawning failed or the RNG check failed):
    - `Fail` sound plays at 0.5 volume
    - Yield for 0.5 seconds
    - If hardmode is true, move 2 (biting attack) is used
