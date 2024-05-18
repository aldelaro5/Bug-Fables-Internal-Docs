# HeavyStrike
Many of the logic here is shared by [Beetle's basic attack](../Basic%20attacks/Beetle.md). As such, it is assumed that the actor is `Beetle`'s player party member.

## [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|25.0|[HoldKeyCountdown](../../Action%20commands/HoldKeyCountdown.md)|{6.0, 1.0}

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1_a|DoCommand 1 set `commandsuccess` to true|`Beetle`'s player party member|`availabletargets[target]`|`Beetle`'s `atk` + 2 unclamped|[Flip](../../Damage%20pipeline/AttackProperty.md#attackproperty)|{[NoSound](../../Damage%20pipeline/DoDamage.md#nosound)}|false|
|1_b|DoCommand 1 set `commandsuccess` to false|`Beetle`'s player party member|`availabletargets[target]`|`Beetle`'s `atk` clamped from 0 to 2. NOTE: This incorrectly removes influences of negative `atk` values|[Flip](../../Damage%20pipeline/AttackProperty.md#attackproperty)|Empty array|false|

## Logic sequence

- If in `demomode`, CreateHelpBox is called with id 1
- Camera is moved to look at targetentity
- targetentity gets `overridejump` set to true
- `Beetle` MoveTowards in front of the enemy (calculated based on its `size` or `freezesize`)
- Yield until `Beetle` is done with its `forcemove`
- Camera zooms in slightly
- `Beetle` animstate set to 103
- DoCommand 1 call happens
- `Charge3` sound played on `sounds[9]` with 0.5 pitch on loop
- Yield until `doingaction` is false (for each yield, the `Charge9` sound pitch is increased by 0.01 and clamped from 0.5 to 1.25 while `Beetle` sprite shakes slightly by random displacement)
- `sounds[9]` (`Charge3` sound) is stopped
- Camera zooms out slightly
- If `commandsuccess` (the action command succeeded):
    - DoDamage 1_a call happens
    - ShakeScreen called with ammount of (0.1, 0.1, 0.1) and time of 0.25
    - `HugeHit` sound is played
    - `Beetle` animstate set to 104
    - BounceEnemy for the targetentity with bounce of 1 and multiplier of 1.6
    - yield until the BounceEnemy is done
    - 0.15 seconds yield
- Otherwise (the action command failed)
    - `AtkFail` sound plays
    - DoDamage 1_b call happens
    - `Beetle` gets `overridejump` and `overrideanim` set to true
    - `Beetle` jumps with an animstate of 106 and a slight -z `spin`
    - 0.3 seconds yielded
    - `Beetle` animstate set to 105 with a very slight +z `spin`
    - Yield until `Beetle` is `onground`
    - `Beetle` animstate set to 107
    - `Prefabs/Particles/deathsmokelow` instantated and destroyed in 1.0 seconds
    - Frame yield
    - `Beetle` x/z angles reset to 0.0 and `spin` zeroed out
    - 0.45 seconds yield
- targetentity gets `overrideanim` set to false
- `Beetle` gets `overridejump` and `overrideanim` set to false
- `Beetle` spin is zeroed out
- targetentity gets `overridejump` set to false