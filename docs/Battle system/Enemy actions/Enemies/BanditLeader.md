# `BanditLeader`

## Special [UseItem](../../Battle%20flow/Action%20coroutines/UseItem.md) logic
This enemy has very special handling whenever the UseItem action coroutine is invoked where they will interupt the item usage (even going through [stopping conditions](../../Actors%20states/IsStopped.md) if they have any).

To learn more, check the [`BanditLeader` special UseItem logic](../../Battle%20flow/Action%20coroutines/UseItem.md#banditleader-specific-logic) for more details.

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the sword slash move, the odds to use the sword thrust move immediately after on the same actor turn changes to 7/10 from 5/10 when [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.5
- In the sword thrust move, the base damage amount is 1 instead of 2 per hit

## Move selection
2 moves are possible:

1. A single target sword thrust that hits multiple times
2. A single target sword slash

The usage of the moves is determined by these odds:

|Move|Odds|
|1|4/7|
|2|3/7|

However, it is possible if move 2 is used that move 1 is used right after in the same actor turn. This happens if all the following conditions are fufilled at the end of move 2:

- [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.5
- A 5/10 RNG check passes (7/10 instead if hardmode is true)

## Pre move logic
The following logic always happen at the start of the actor turn if [HPPercent](../../Actors%20states/HPPercent.md) is 0.65 or less. Notably, `moves` is set to 2 and `cantmove` decremented which gives 2 actor turns per main turn on this enemy including the current one:

- animstate set to 103
- `moves` set to 2
- `cantmove` decremented
- Yield for 0.35 seconds
- `StatUp` sound plays
- [StatEffect](../../Visual%20rendering/StatEffect.md) called with type 5 (yellow up arrow)
- Yield for 0.5 seconds

## Post move logic
The following logic always happen at the end of the action:

- If `cantmove` is higher than -1 (meaning this is the last actor turn they can do) and a 50% RNG check passes:
    - [isdefending](../../Actors%20states/Enemy%20features.md#isdefending) set to true (activate their guard)
    - [defenseonhit](../../Actors%20states/Enemy%20features.md#defenseonhit-and-isdefending) set to 2
    - The local startstate is set to 24 (`Block`)
- Otherwise:
    - [isdefending](../../Actors%20states/Enemy%20features.md#isdefending) set to false (gets out of their guard)
    - [defenseonhit](../../Actors%20states/Enemy%20features.md#defenseonhit-and-isdefending) set to 0
    - The local startstate is set to 0 (`Idle`)

## Move 1 - Sword thrust
A single target sword thrust that hits multiple times

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen 2 times (random between 2 and 3 times instead if [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.65)|This enemy|The selected `playertargetID`|2 (1 instead if hardmode is true)|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- `Gleam` particles plays with `Gleam` sound at this enemy + (0.1, 2.75, -0.1) with 0.5 alivetime
- Yield for 0.5 seconds
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playertargetentity` position + (2.5, 0.0, -0.1)
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (2.5, 0.0, -0.1) with 2.0 muiltiplier using 1 (`Walk`) as walkstate and 0 (`Idle`) as stopstate
- Yield until `forcemove` is done
- The amount of hits is determined. It's always 2 unless [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.65 where it's random between 2 and 3 times
- For each hit:
    - `Woosh` sound plays with a pitch of 1.0 on the first and third hit or 1.1 on the second hit
    - animstate set to 105 on the first and third hit or 106 on the second hit
    - Yield for 0.15 seconds
    - DoDamage 1 call happens
    - `Slash2` sound happens
    - yield for 0.45 seconds
- Yield for 0.5 seconds

## Move 2 - Sword slash
A single target sword slash.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|3|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playertargetentity` position + (2.5, 0.0, -0.1)
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (2.5, 0.0, -0.1) with 2.0 muiltiplier using 1 (`Walk`) as walkstate and 0 (`Idle`) as stopstate
- Yield until `forcemove` is done
- animstate set to 103
- `Slash2` sound plays
- Yield for 0.85 seconds
- animstate set to 107
- `Slash` sound plays
- DoDamage 1 call happens
- Yield for 0.65 seconds
- If [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.5 and a 5/10 RNG test passes (7/10 instead if hardmode is true):
    - `Gleam` particles plays with `Gleam` sound at this enemy + (-0.2, 2.75, -0.1) with 0.5 alivetime
    - Move 1 (sword thrust) is used immeditaly on the same actor turn, but without the logic of everything until the `forcemove` yields as the enemy is already positioned with the right target selected
