# `SkullCaterpillar`

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: This is a cooldown for the usage of the screaming attack move. See the move selection section for details on how this cooldown works as it doesn't behave like a typical actor turn cooldown

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the rolling attack move, the amount of frames this takes to move changes to 28.0 frames from 33.0 frames

## Move selection
2 moves are possible:

1. A party wide rolling attack
2. A party wide screaming attack

The decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|3/5|
|2|2/5|

However, move 2 won't be used if `data[0]` (its usage cooldown) hasn't expired yet. If this happens, move 1 is used instead. 

The cooldown is set to expire after either 1 or 2 decrements determined randomly. Move 1 always causes a decrement when used, but move 2 will too if it is selected and the cooldown hasn't expired yet. This means that effectively:

- If it was set to expire after 1 decrement, move 2 becomes usable after the next actor turn no matter which move is selected and used
- If it was set to expire after 2 decrements, then it depends on which move is selected on the next actor turn:
    - If it's move 1, then move 2 still won't be usable on the next actor turn
    - If it's move 2, it will become usable on the next actor turn

## Move 1 - Rolling attack
A party wide rolling attack.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|2|null|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence

- `Curl` sound plays
- Camera moves to look near this enemy
- animstate set to 100
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on this enemy
- Yield for 0.5 seconds
- Yield all frames until this enemy is `onground`
- `Spin2` sound plays on loop using `sounds[9]` with 1.25 pitch
- ShakeSprite called with an intensity of (0.1, 0.0, 0.0) and 45.0 frametimer
- Yield for 0.75 seconds
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `sounds[9]` pitch set to 1.1
- Over the course of 33.0 frames (28.0 frames instead if hardmode is true), this enemy moves to (-15.0, 0.0, 0.0) via a lerp. On the first frame its x position becomes lower than 4.5:
    - ShakeScreen called with 0.15 ammount and 0.8 time
    - PartyDamage 1 call
- Over the course of 41.0 frames, this enemy moves to startp via a lerp
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on this enemy
- Yield for 0.5 seconds
- Yield all frames until this enemy is `onground`
- `data[0]` is decremented if it is above 0

## Move 2 - Screaming attack
A party wide screaming attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|2|[Numb](../../Damage%20pipeline/AttackProperty.md)|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence

- `checkingdead` set to a new ScreamWaves (`checkingdead` is set to null when completed)
- Yield all frames until `checkingdead` is null
- `data[0]` is set to either 1 or 2 determined randomly

This is what the coroutine ends up doing:

- Camera moves to look near this enemy
- animstate set to 101
- ShakeSprite called with an intensity of (0.05, 0.0, 0.0) and 85.0 frametimer
- Yield for 0.85 seconds
- `Gleam` particles plays with the `Gleam` sound at this enemy's `sprite` + its [CenterPos](../../Actors%20states/CenterPos.md) + (0.5, 1.0, -0.1) with 0.5 alivetime
- Yield for 0.5 seconds
- Done 5 times in a separate coroutine started in paralel called MainManager.Waves:
    - A `Prefabs/Objects/SphereGlowEffect` GameObject is instantiated rooted at this enemy's world [CenterPos](../../Actors%20states/CenterPos.md) with a scale of 0.0x
    - Over the course of 31.0 frames, the scale of `Prefabs/Objects/SphereGlowEffect` changes to (50.0, 50.0, 1.0) via a lerp
    - The `Prefabs/Objects/SphereGlowEffect` gets destroyed
    - Yield for 0.25 seconds
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `MadesphyScream` sound plays
- animstate set to 103
- ShakeScreen called with an ammount of 0.2 and 1.0 time
- Yield for 0.25 seconds
- PartyDamage 1 call happens
- Yield for a second
- `checkingdead` set to null which signals the caller that this coroutine completed
