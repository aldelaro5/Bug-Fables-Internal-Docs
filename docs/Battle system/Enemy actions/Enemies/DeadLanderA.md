# `DeadLanderA`

## BattleControl.[GetEXP](../../../TextAsset%20Data/Enemies%20data.md#exp-logic) special logic
This enemy is part of the set of the enemies that yields a different clamping on the rewarded amount of exp given their scaled `exp` field when they die (processed by [CheckDead](../../Battle%20flow/Action%20coroutines/CheckDead.md)).

If all of these conditions are fufilled, the rewarded amount of exp is clamped to 20 instead of 15 like most other enemies:

- [flags](../Flags%20arrays/flags.md) 613 is false (RUIGEE is inactive)
- `partylevel` is less than 27 (not yet at max rank)
- [Flags](../Flags%20arrays/flags.md) 166 is false (not during an EX mode B.O.S.S session)

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The odds to dig and set [position](../../Actors%20states/BattlePosition.md) to `Underground` in the post move logic changes to 40% from 30%
- In the goo projectiled throw move, the speed changes to be random between 20 and 25 inclusive cast to float instead of being random between 20 and 31 inclusive cast to float

## Move selection
2 moves are possible:

1. A multiple targets goo projectiles throw
2. A single target bite attack

Move 2 is always used if all of the following conditions are fufilled:

- `firststrike` is false (the enemy party isn't performing their first turn advantage)
- This enemy [position](../../Actors%20states/BattlePosition.md) wasn't `Underground` at the start of the action (before the pre move logic)
- A 50% RNG check passes

If the above aren't fufilled, move 1 is always used instead.

## Pre move logic
The following logic always happen if [position](../../Actors%20states/BattlePosition.md) is `Underground` to change it to `Ground`:

- `DigPop3` sound plays
- `checkingdead` is set to a new [ChangePosition](../ChangePosition.md) call to change the [position](../../Actors%20states/BattlePosition.md) of this enemy to `Ground`
- Yield all frames until `checkingdead` is null (the coroutine completed)

## Post move logic
The following logic always happen after using a move if a 30% RNG check passes (40% instead if hardmode is true):

- animstate set to 100
- `Dig3` sound plays
- `checkingdead` set to a new [Dig](../Dig.md) call with 0 jump to set this enemy's [position](../../Actors%20states/BattlePosition.md) to `Underground`
- Yield all frames until `checkingdead` is null (the coroutine completed)

## Move 1 - Goo projectiles throw
A multiple targets goo projectiles throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen 3 times, but each calls requires that at least 1 player party member is alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|3|[Poison](../../Damage%20pipeline/AttackProperty.md) or [Numb](../../Damage%20pipeline/AttackProperty.md) determined randomly with uniform odds|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) (target changes for each calls)||A new sprite object rooted using the `projectilepsrites[15]` sprite (an inverted honey projectile) positioned at this enemy + (-1.6, 1.0, -0.1) with a and material color of pure yellow with a ShadowLite SetUp with 0.3 opacity and 0.5 size|Random between 20 and 31 inclusive then cast to float (between 20 and 25 inclusive then cast to float instead if harmode is true)|0.0|`keepcolor`|`ElecFast`|`BubbleBurst`|null|Vector3.zero|false|

### Logic sequence

- Camera moves to look near the midpoint between this enemy and `partymiddle`
- A new SpriteRenderer array is created with 3 as the length to hold each projectile object
- Done 3 times:
  - If at least 1 player party member is alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)):
      - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
      - `DLAlphaGoo1` sound plays
      - animstate set to 103
      - Yield for a second
      - The projectile element is set to a new sprite object rooted using the `projectilepsrites[15]` sprite (an inverted honey projectile) positioned at this enemy + (-1.6, 1.0, -0.1) with a and material color of pure yellow with a ShadowLite SetUp with 0.3 opacity and 0.5 size
      - `DLAlphaGoo2` sound plays
      - Projectile 1 call happens
      - animstate set to 104
      - Yield for 0.5 seconds
- Yield all frames until all projectiles objects are null (all Projectile 1 calls completed)
- Yield for 0.5 seconds

## Move 2 - Bite attack
A single target bite attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|4|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- animstate set to 100
- Yield for 0.25 seconds
- y `spin` set to 30.0
- `Dig3` sound plays
- ChangeScale called to change the scale to 0.0x with 30.0 framtime with force
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- position set to `playertargetentity` position + (2.0, 0.0, -0.1)
- ChangeScale called to change the scale to the `startscale` value before this action with 30.0 framtime with force
- `DigPop` sound plays
- Yield for 0.5 seconds
- FliAngle called with setangle
- `spin` zeroed out
- animstate set to 101
- `DLAlphaChomp` sound plays
- Yield for a random interval between 0.6 and 0.8 seconds
- animstate set to 102
- `Bite` sound plays with 1.2 pitch
- DoDamage 1 call happens
- Yield for 0.5 seconds
- animstate set to 100
- `DLAlphaWoosh` sound plays
- Yield for 0.25 seconds
- y `spin` set to 30.0
- `Dig3` sound plays
- ChangeScale called to change the scale to 0.0x with 30.0 framtime with force
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- Yield for a frame
- position changed to startp
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `DigPop3` sound plays
- ChangeScale called to change the scale to the `startscale` value before this action with 30.0 framtime with force
- Yield for 0.5 seconds
- FliAngle called with setangle
- `spin` zeroed out
