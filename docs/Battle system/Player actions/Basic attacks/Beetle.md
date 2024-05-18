# `Beetle`'s basic attack

## [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|20.0|[HoldKeyCountdown](../../Action%20commands/HoldKeyCountdown.md)|{4.0, 1.0}

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1_a|DoCommand 1 set `commandsuccess` to true|`Beetle`'s player party member|`availabletargets[target]`|`Beetle`'s `atk` unclamped|[Flip](../../Damage%20pipeline/AttackProperty.md#attackproperty)|null|false|
|1_b|DoCommand 1 set `commandsuccess` to false while `DoublePain` is unequipped and HARDEST isn't active|`Beetle`'s player party member|`availabletargets[target]`|`Beetle`'s `atk` clamped from 0 to 1. NOTE: This incorrectly removes influences of negative `atk` values|[Flip](../../Damage%20pipeline/AttackProperty.md#attackproperty)|{[FailSound](../../Damage%20pipeline/DoDamage.md#failsound)}|false|
|1_c|DoCommand 1 set `commandsuccess` to false while `DoublePain` is equipped or HARDEST is active|`Beetle`'s player party member|`availabletargets[target]`|`Beetle`'s `atk` / 2 floored, unclamped. NOTE: This incorrectly increases `atk` when it is negative. Additionally, it can also result in a higher value than 1_b's damageammount if the `atk` value is high enough which is unexpected|null|{[FailSound](../../Damage%20pipeline/DoDamage.md#failsound)}|false|

## Logic sequence

- If in `demomode`, CreateHelpBox is called with id 1
- Camera is moved to look at targetentity
- targetentity gets `overridejump` set to true
- `Beetle` MoveTowards in front of the enemy (calculated based on its `size` or `freezesize`)
- Yield until `Beetle` is done with its `forcemove`
- Camera zooms in slightly
- `Beetle` animstate set to 103
- DoCommand 1 call happens
- Yield until `doingaction` is false
- Camera zooms out slightly
- If `commandsuccess` (the action command succeeded):
    - DoDamage 1_a call happens
    - `Beetle` animstate set to 104
    - BounceEnemy for the targetentity with bounce of 0 and multiplier of 1.0
    - yield until the BounceEnemy is done
    - 0.15 seconds yield
- Otherwise (the action command failed)
    - If [flags](../../../Flags%20arrays/flags.md) 614 is false (HARDEST is inactive) and `DoublePain` [medal](../../../Enums%20and%20IDs/Medal.md) is unequipped, DoDamage 1_b call happens
    - Otherwise (on hard mode or HARDEST), DoDamage 1_c call happens
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