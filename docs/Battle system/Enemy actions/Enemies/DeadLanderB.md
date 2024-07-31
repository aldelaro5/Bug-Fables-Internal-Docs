# `DeadLanderB`

## BattleControl.[GetEXP](../../../TextAsset%20Data/Enemies%20data.md#exp-logic) special logic
This enemy is part of the set of the enemies that yields a different clamping on the rewarded amount of exp given their scaled `exp` field when they die (processed by [CheckDead](../../Battle%20flow/Action%20coroutines/CheckDead.md)).

If all of these conditions are fufilled, the rewarded amount of exp is clamped to 20 instead of 15 like most other enemies:

- [flags](../../../Flags%20arrays/flags.md) 613 is false (RUIGEE is inactive)
- `partylevel` is less than 27 (not yet at max rank)
- [Flags](../../../Flags%20arrays/flags.md) 166 is false (not during an EX mode B.O.S.S session)

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the aerial slam attack, the frametime of the [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) call before the DoDamage call changes to 25.0 fram 32.0

## Move selection
2 moves are possible:

1. A single target vomit attack that hits twice
2. A party wide aerial slam attack

The decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|50%|
|2|50%|

## Move 1 - Vomit attack
A single target vomit attack that hits twice.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen 2 times|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|[Poison](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near this enemy
- animstate set to 100
- `bobspeed` set to 0.0
- ShakeSprite called with 0.1 intensity and 90.0 frametimer
- `DLBetaCharge` sound plays
- Yield for a second
- Yield for 0.5 seconds
- `DLBetaDing` sound plays
- animstate set to 101
- `DLBetaVomit` sound plays
- A new sprite object is created childed to `sprite` using the `Sprites/Entities/deadlandera` index 57 sprite (vomit) positioned at (-0.8, 0.5, -0.1) with a scale of (0.015 * the distance between this enemy and `playertargetentity`, 0.0, 1.0)
- `playertargetentity` animstate set to 11 (`Hurt`)
- `sprite` gets an AimConstraint added with an aimVector of (0.0, 1.0, 1.0), a constraintActive of true, a rotationOffset of (0.0, 87.0, -50.0) and a source constraint added with 1.0 weight and `playertargetentity` sourceTransform
- FlipSpriteBool called on the vomit sprite without x, with y every 3.0 frames during -1.0 frames (infinite)
- Camera moves to look near the midpoint between this enemy and `playertargetentity`
- The vomit sprite has GradualScale called to change the scale to (0.015 * the distance between this enemy and `playertargetentity`, 1.0, 1.0) with 5.0 frametime without destroy
- Yield for 0.1 seconds
- Yield for 0.1 seconds
- Done 2 times:
    - If `commandsuccess` is true (blocking), `playertargetentity` animstate is set to 24 (`Block`)
    - DoDamage 1 call happens
    - Yield for 0.5 seconds
    - `playertargetentity` animstate set to 11 (`Hurt`)
    - Yield for 0.25 seconds
- The vomit sprite has GradualScale called to change the scale to (0.015 * the distance between this enemy and `playertargetentity`, 0.0, 1.0) with 30.0 frametime without destroy
- Yield for 0.5 seconds
- The FlipSpriteBool coroutine is stopped
- Yield a frame
- The vomit sprite gets destroyed
- The AimConstraint on `sprite` gets destroyed
- `sprite` angles reset to Vector3.zero
- animstate set to 102
- Yield for 0.5 seconds
- `bobspeed` set to `startbs`

## Move 2 - Slam attack
A party wide aerial slam attack.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|5|null|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence

- `DLFlyOver` sound plays
- `bobspeed` set to 0.0
- Camera moves to look near (-4.5, 1.5, 1.25)
- [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) to `partymiddle` + 5.0 in y with 70.0 frametime using 100 as movestate and 103 as stopstate
- `DLBetaSpike` sound plays
- Yield for 0.5 seconds
- `height` increased by 0.5
- animstate set to 103
- Yield all frames until `forcemoving` is done
- ShakeSprite called with 0.1 intensity and 90.0 frametimer
- Yield for 0.5 seconds
- `Fall2` sound plays
- `startscale` set to 1.25x
- Yield for a frame
- z `spin` set to 15.0
- [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) to `partymiddle` + (0.0, 0.0 - `sprite` local y position + 0.75, -0.2) with 32.0 frametime (25.0 instead if hardmode is true) using 103 as movestate and 103 as stopstate
- animstate set to 103
- Yield all frames until `forcemoving` is done
- `Thud3` sound plays
- `spin` zeroed out
- ShakeScreen called with an ammount of 0.1 and 0.75 time
- `impactsmoke` particles plays at `partymiddle`
- PartyDamage 1 plays
- `startscale` set to (1.25, 0.75, 1.0)
- `sprite` local scale set to `startscale`
- `sprite` angles reset to Vector3.zero
- `sprite` gets a SpriteBounce added SetUp with a freq of 0.75 and spd of 0.5
- A Gradual call starts on the SpriteBounce with a freq of 0.0, 0.0 spd and 45.0 frametime
- Yield for 1 second
- The SpriteBounce is destroyed
- position set to `partymiddle`
- `startscale` reset to 1.0x
- `sprite` scale reset to 1.0x
- `height` set to its value before this action
- `bobspeed` set to `startbs`
- `DLBetaFlyBack` sound plays
