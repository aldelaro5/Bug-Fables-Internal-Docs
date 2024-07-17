# `PeacockSpider`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- At the start of the action, if this enemy is the last `enemydata` remaining, the enemy they will summon is always going to be a [DivingSpider](DivingSpider.md) instead of only being one when [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.5 and being a [JumpingSpider](JumpingSpider.md) when it is 0.5 or above

## Move selection
6 moves are possible:

1. Heals the enemy party member this enemy summoned
2. Inflicts an infinite [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on the enemy party member this enemy summoned
3. Inflicts an infinite [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition on the enemy party member this enemy summoned
4. Sets the `charge` of the enemy party member this enemy summoned to 2
5. Sets the `moves` of the enemy party member this enemy summoned to 2 which grants them 2 actor turns per main turns (excluding the current main turn)
6. A single target triple slash attack

The decision of which move to use is based on odds, but each moves except move 6 has an additional requirement that must be fufilled for the move to be used after it is selected. If the requirement isn't fufilled, the move is rerolled with the same odds. If the requirement is still failed after the second attempt, move 6 will be used. Here are the odds and requirements of each moves:

|Move|Odds|Requirement|
|---:|----|-----------|
|1|3/12|[HPPercent](../../Actors%20states/HPPercent.md) is 0.65 or less|
|2|2/12|The enemy party member this enemy summoned doesn't already have the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition|
|3|2/12|The enemy party member this enemy summoned doesn't already have the [DefenseUp](../../Actors%20states/BattleCondition/AttackUp.md) condition|
|4|2/12|The enemy party member this enemy summoned has a `charge` of 0|
|5|2/12|The enemy party member this enemy summoned has a `moves` of 1|
|6|1/12|None, the move is always used when selected|

## Pre move logic
There is logic that is always performed at the start of the action.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Happens only if the summoned enemy [IsStopped](../../Actors%20states/IsStopped.md)|null|The summoned enemy party member|1 (this cannot be lethal to the target as enforced after the call)|[NoExceptions](../../Damage%20pipeline/AttackProperty.md)|null|false|

### Logic Sequence

- If this enemy is the last `enemydata` remaining, an enemy will be summoned:
    - animstate set to 101
    - `PeacockSpiderNPCSummon` sound plays
    - `checkingdead` is set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon an enemy with the type `Offscreen` at Vector3.zero. The enemy to summon is always a [DivingSpider](DivingSpider.md) unless hardmode is false and [HPPercent](../../Actors%20states/HPPercent.md) is 0.5 or above where the enemy is a [JumpingSpider](JumpingSpider.md)
    - Yield all frames until `checkingdead` is null (the coroutine completed)
    - `PeacockSpiderNPCSummonSuccess` sound plays
    - animstate set to 0 (`Idle`)
    - `flip` set to false
    - [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on this enemy
    - Yield for 0.5 seconds
    - The `exp` of the summoned enemy is clamped from 0 to 3
- The first enemy party member who isn't this enemy is obtained which should always be the one that was just summoned or already summoned before and no one else. This will be used throughout the action
- If the summoned enemy [IsStopped](../../Actors%20states/IsStopped.md), they will be cured of their condition with the following:
    - `PeacockSpiderSlapAtkRaise` sound plays
    - animstate set to 5 (`Angry`)
    - Yield for 0.5 seconds
    - `PeacockSpiderSlapAtk` sound plays
    - animstate set to 102
    - [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) called on the summoned enemy
    - DoDamage 1 call happens
    - If the summoned enemy's `hp` is 0 or below, it is set to 1 (meaning DoDamage 1 cannot be lethal to them)
    - The summoned enemy's `cantmove` is set to 0 meaning they can act on this main turn after this enemy's action is done
    - Yield for a second

## Move 1 - Heal the summoned enemy
Heals the enemy party member this enemy summoned. No damages are dealt.

### Logic sequence

- A new `Prefabs/Particles/MagicConstant` GameObject is created rooted at this enemy position with a scale of 2.0x
- `PeacockSpiderNPCBuff` sound plays
- `Prefabs/Particles/MagicConstant`'s ParticleSystem's MainModule's startColor is set to a constant color curve with a value of pure red with 0.5 opacity
- animstate set to 103
- Camera moves to loop between this enemy and the summoned enemy with a bias towards this enemy
- Yield for a second
- Yield for 0.5 seconds
- [Heal](../../Actors%20states/Heal.md) called to heal the summoned enemy to an amount of `hp` equal to their `maxhp` * 0.15 ceiled
- `Prefabs/Particles/MagicConstant` moved offscreen at -9999.0 in y then destroyed in 2.0 seconds
- Yield for 1 second
- `flip` set to false

## Move 2 - Infinite [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) on the summoned enemy
Inflicts an infinite [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on the enemy party member this enemy summoned. No damages are dealt.

### Logic sequence

- A new `Prefabs/Particles/MagicConstant` GameObject is created rooted at this enemy position with a scale of 2.0x
- `PeacockSpiderNPCBuff` sound plays
- `Prefabs/Particles/MagicConstant`'s ParticleSystem's MainModule's startColor is set to a constant color curve with a value of FF4C00 (mostly red) with 0.5 opacity
- animstate set to 100
- Camera moves to loop between this enemy and the summoned enemy with a bias towards this enemy, but zoomed in
- Yield for a second
- Yield for 0.5 seconds
- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on the summoned enemy for 999999 main turns (basically infinite) with effect
- `Prefabs/Particles/MagicConstant` moved offscreen at -9999.0 in y then destroyed in 2.0 seconds
- Yield for 1 second
- `flip` set to false

## Move 3 - Infinite [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) on the summoned enemy
Inflicts an infinite [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition on the enemy party member this enemy summoned. No damages are dealt.

### Logic sequence

- A new `Prefabs/Particles/MagicConstant` GameObject is created rooted at this enemy position with a scale of 2.0x
- `PeacockSpiderNPCBuff` sound plays
- `Prefabs/Particles/MagicConstant`'s ParticleSystem's MainModule's startColor is set to a constant color curve with a value of pure blue with 0.5 opacity
- animstate set to 100
- Camera moves to loop between this enemy and the summoned enemy with a bias towards this enemy, but zoomed in
- Yield for a second
- Yield for 0.5 seconds
- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition on the summoned enemy for 999999 main turns (basically infinite) with effect
- `Prefabs/Particles/MagicConstant` moved offscreen at -9999.0 in y then destroyed in 2.0 seconds
- Yield for 1 second
- `flip` set to false

## Move 4 - `charge` of the summoned enemy set to 2
Sets the `charge` of the enemy party member this enemy summoned to 2. No damages are dealt.

### Logic sequence

- A new `Prefabs/Particles/MagicConstant` GameObject is created rooted at this enemy position with a scale of 2.0x
- `PeacockSpiderNPCBuff2` sound plays
- `Prefabs/Particles/MagicConstant`'s ParticleSystem's MainModule's startColor is set to a constant color curve with a value of pure green with 0.5 opacity
- animstate set to 101
- Camera moves to loop between this enemy and the summoned enemy with a bias towards this enemy, but zoomed in
- Yield for a second
- Yield for 0.5 seconds
- The summoned enemy's `charge` is set to 2
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on the summoned enemy with type 4 (green up arrow)
- `Prefabs/Particles/MagicConstant` moved offscreen at -9999.0 in y then destroyed in 2.0 seconds
- Yield for 1 second
- `flip` set to false

## Move 5 - `moves` of the summoned enemy set to 2
Sets the `moves` of the enemy party member this enemy summoned to 2 which grants them 2 actor turns per main turns (excluding the current main turn). No damages are dealt.

### Logic sequence

- A new `Prefabs/Particles/MagicConstant` GameObject is created rooted at this enemy position with a scale of 2.0x
- `PeacockSpiderDoubleTurnDance` sound plays
- `Prefabs/Particles/MagicConstant`'s ParticleSystem's MainModule's startColor is set to a constant color curve with a value of FFBF00 (mostly orange) with 0.5 opacity
- animstate set to 101
- Camera moves to loop between this enemy and the summoned enemy with a bias towards this enemy, but zoomed in
- Yield for a second
- Yield for 0.5 seconds
- The summoned enemy's `moves` is set to 2 which grants them 2 actor turns per main turns (excluding the current main turn)
- `StatUp` sound plays
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on the summoned enemy with type 5 (yellow up arrow)
- `Prefabs/Particles/MagicConstant` moved offscreen at -9999.0 in y then destroyed in 2.0 seconds
- Yield for 1 second
- `flip` set to false

## Move 6 - Triple slash attack
A single target triple slash attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen 3 times|This enemy|The selected `playertargetID`|2|null|null|`commandsuccess`|

### Logic sequence
It is possible that a new `Prefabs/Particles/MagicConstant` GameObject is created rooted at this enemy position with a scale of 2.0x if this move is used after failing to use another move twice in a row. If this is the case, the GameObject gets destroyed first.

Here's the rest of the move logic:

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (2.0, 0.0, -0.1) with 2.0 multiplier
- Yield all frames until `forcemove` is done
- `PeacockSpiderSlapAtkRaise` sound plays
- animstate set to 5 (`Angry`)
- `Gleam` particles plays with the `Gleam` sound at this enemy + (0.2, 1.5, -0.2) with 0.5 alivetime
- Yield for 0.5 seconds
- Done 3 times:
    - `PeacockSpiderSlapAtk` sound plays
    - animstate set to 102 (except for the second hit where it's set to 104 instead)
    - `model` y angle set to 30.0 (except for the second hit where it's set to -30.0 instead)
    - DoDamage 1 call happens
    - Camera zooms in a bit
    - Yield for 0.5 seconds
    - Yield for 0.1 seconds
- `model` angles reset to Vector3.zero
- `flip` set to false
