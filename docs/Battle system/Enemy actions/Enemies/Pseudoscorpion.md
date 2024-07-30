# `Pseudoscorpion`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the pincer slash attack move, there's now a 6/10 chance for the enemy to do a fake slash animation before the real one. This will be will be indicated by jumping 2 times at the start of the move. If they do that, the DoDamage will have the [sleep](../../Damage%20pipeline/AttackProperty.md) property as opposed to be [poison](../../Damage%20pipeline/AttackProperty.md)

## Move selection
3 moves are possible:

1. A single target pincer slash attack
2. Changes their [position](../../Actors%20states/BattlePosition.md) to `Underground` by digging
3. A single target undergound strike attack

Move 3 is always (and only) used if [position](../../Actors%20states/BattlePosition.md) is `Underground`.

As for the other moves, the decision of which moves to use depends on the following odds:

|Move|Odds|
|---:|----|
|1|3/5|
|2|2/5|

## Move 1 - pincer slash attack
A single target pincer slash attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|2|If hardmode is false, it's [poison](../../Damage%20pipeline/AttackProperty.md). If it's true, it has 6/10 chance to be [sleep](../../Damage%20pipeline/AttackProperty.md) instead (4/10 chance to still be [poison](../../Damage%20pipeline/AttackProperty.md))|{[BlockSoundOnly](../../Damage%20pipeline/DoDamage.md#blocksoundonly)}|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- This is where the 2 jumps may happen. It will if hardmode is true and a 6/10 RNG check passes (also makes the property of the Damage 1 call [sleep](../../Damage%20pipeline/AttackProperty.md)). This is what happens 2 times if that is the case:
    - `Jump` sound plays with 1.1 pitch and 0.8 volume
    - [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
    - Yield for 0.33 seconds
    - Yield all frames until `onground` is true
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` + (1.25, 0.0, -0.1). The camera moves towards this point too
- Yield all frames until `forcemove` is done
- animstate set to 100
- `Chew` sound plays
- Yield for 0.6 seconds
- If the 2 jumps happened earlier, a fake attack animation is done:
    - `Select0` sound plays at 0.6 volume
    - animstate set to 104
    - Yield for 0.5 seconds
    - animstate set to 100
    - Yield for 0.6 seconds
- animstate set to 102
- DoDamage 1 call happens
- `Hit2` sound plays
- Yield for 0.75 seconds

## Move 2 - Changes their [position](../../Actors%20states/BattlePosition.md) to `Underground`
Changes their [position](../../Actors%20states/BattlePosition.md) to `Underground` by digging. No damages are dealt.

### Logic sequence

- `checkingdead` set to a new [Dig](../Dig.md) call on this enemy
- Yield all frames until `checkingdead` goes to null (the Dig coroutine completed)

## Move 3 - Undergound strike attack then change [position](../../Actors%20states/BattlePosition.md) to `Ground`
A single target undergound strike attack. [position](../../Actors%20states/BattlePosition.md) is set to `Ground` after.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `checkingdead` is set to a new UndergroundStrike coroutine with 3 damage, this enemy, no properties without staying underground
- Yield all frames until `checkingdead` goes to null (the coroutine is done)

This is effectively what the coroutine does:

- Camera moves to look near `playerdata[playertargetID]`, but zoomed in
- `Digging` sound plays on loop using `sounds[9]` with 1.1 pitch and 0.75 volume
- Over the course of 100.0 frames, this enemy moves to `playertargetentity` position + -0.25 in z
- Yield for 0.25 seconds
- `sounds[9]` stopped
- `DigPop` sound plays
- DoDamage 1 call happens
- y `spin` set to 20.0
- `digging` set to false
- scale reset to `startscale`
- animstate set to 103
- `DirtExplodeLight` particle plays at this enemy position
- ShakeScreen with 0.15 ammount and 0.2 time
- Over the course of 50.0 frames, position moves to `playertargetentity` + 2.0 in x using a BeizierCurve3 with a ymax of 6.0
- `spin` zeroed out
- [position](../../Actors%20states/BattlePosition.md) set to `Ground`
- Yield for 0.25 seconds
- `checkingdead` set to null to signal DoAction that the coroutine completed
