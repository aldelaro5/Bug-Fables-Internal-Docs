# `WaspGeneral`

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: This is set to 1 the first time the triple slap move is used and it prevents some animation logic to occur for the rest of the battle when the move is used again

## Move selection
5 moves are possible:

1. Grants an [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition to the first `enemydata` other than themselves who didn't already had it for 3 main turns
2. Grants a [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition to the first `enemydata` other than themselves who didn't already had it for 3 main turns
3. Heals the first `enemydata` other than themselves who has an [HPPercent](../../Actors%20states/HPPercent.md) less than 0.6 by 4 `hp`
4. Does nothing
5. A single target triple slap attack

Move 5 is always (and only) used if this enemy is the last `enemydata` remaining.

As for the other moves, the decision of which move to use is based on odds. However, with the exception of move 4, each move has an additional requirement that must be fufilled in order for the move to be used after it is selected. If the requirement is not fufilled and the move is selected, move 4 will be used instead. Here are the odds and requirements:

|Move|Odds|Requirment|
|---:|----|---------|
|1|2/8|At least one `enemydata` other than this enemy doesn't already have the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition|
|2|2/8|At least one `enemydata` other than this enemy doesn't already have the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition|
|3|3/8|At least one `enemydata` other than this enemy has an [HPPercent](../../Actors%20states/HPPercent.md) lower than 0.6|
|4|1/8|None, the move is always used|

## Move 1 - Grants an [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition to another enemy party member
Grants an [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition to the first `enemydata` other than themselves who didn't already had it for 3 main turns. No damages are dealt.

### Logic sequence

- The receiver of the heal is determined. It's the first `enemydata` other than themselves who didn't already have the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition
- animstate set to 103
- Yield for 0.6 seconds
- A new sprite object is created rooted using the `VitalitySeed` [item](../../../Enums%20and%20IDs/Items.md) sprite positioned at this enemy + (-1.0, 2.0, -0.1) and destroyed in 0.75 seconds
- animstate set to 104
- `ItemHold` sound plays
- Yield for 0.75 seconds
- animstate set to 0 (`Idle`)
- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) is called to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on the enemy party member determined earlier for 3 main turns with effect
- Yield for 0.5 seconds

## Move 2 - Grants a [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition to another enemy party member
Grants a [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition to the first `enemydata` other than themselves who didn't already had it for 3 main turns. No damages are dealt.

### Logic sequence

- The receiver of the heal is determined. It's the first `enemydata` other than themselves who didn't already have the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition
- animstate set to 103
- Yield for 0.6 seconds
- A new sprite object is created rooted using the `GenerousSeed` [item](../../../Enums%20and%20IDs/Items.md) sprite positioned at this enemy + (-1.0, 2.0, -0.1) and destroyed in 0.75 seconds
- animstate set to 104
- `ItemHold` sound plays
- Yield for 0.75 seconds
- animstate set to 0 (`Idle`)
- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) is called to inflict the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition on the enemy party member determined earlier for 3 main turns with effect
- Yield for 0.5 seconds

## Move 3 - Heals another enemy party member `hp`
Heals the first `enemydata` other than themselves who has an [HPPercent](../../Actors%20states/HPPercent.md) less than 0.6 by 4 `hp`. No damages are dealt.

### Logic sequence

- The receiver of the heal is determined. It's the first `enemydata` other than themselves who has an [HPPercent](../../Actors%20states/HPPercent.md) less than 0.6
- animstate set to 103
- Yield for 0.6 seconds
- A new sprite object is created rooted using the `CrunchyLeaf` [item](../../../Enums%20and%20IDs/Items.md) sprite positioned at this enemy + (-1.0, 2.0, -0.1) and destroyed in 0.75 seconds
- animstate set to 104
- `ItemHold` sound plays
- Yield for 0.75 seconds
- animstate set to 0 (`Idle`)
- [Heal](../../Actors%20states/Heal.md) is called on the enemy party member determined earlier to heal their `hp` by 4
- Yield for 0.5 seconds

## Move 4 - Does nothing
Does nothing. No damages are dealt.

### Logic sequence

- animstate set to 101
- Yield for 0.75 seconds

## Move 5 - Triple slap attack
A single target triple slap attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen 3 times|This enemy|The selected `playertargetID`|1|null|null|`commandsuccess`|

### Logic sequence

- If `data[0]` is 0 (this move wasn't used before):
    - `Jump` sound plays with 1.1 pitch
    - [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) is called on this enemy
    - `basestate` and animstate set to 13 (`BattleIdle`)
    - `walkstate` set to 23 (`Chase`)
    - Yield for 0.75 seconds
    - `data[0]` is set to 1 (meaning this logic won't happen again when this moved is used again)
    - A new `Prefabs/Particles/Tired` GameObject is instantiated chiled to this enemy with a local position of `cursoroffset` - 0.5 in y with angles of (-130.0, 270.0, 0.0)
- If `inevent` is false (no [EventDialogue](../../Battle%20flow/EventDialogue.md) is ongoing):
    - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
    - Camera moves to look near `playerdata[playertargetID]`
    - [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (1.25, 0.0, -0.1) with a 2.0 multiplier using 23 (`Chase`) as walkstate and 100 as stopstate
    - Yield all frames until `forcemove` is done
    - `UltimaxAtk` sound plays
    - Done 3 times:
        - DoDamage 1 call happens
        - Yield for 0.65 seconds
    - [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
    - [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` startp with a 2.0 multiplier using 23 (`Chase`) as walkstate and the local startstate as stopstate
    - Yield all frames until `forcemove` is done
