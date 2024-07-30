# `Midge`

## Assumptions
It is assumed that this enemy is loaded with a [chargeonotherenemy](../../Actors%20states/Enemy%20features.md#chargeonotherenemy) configured so their `hitaction` logic works (typically it would be set to `Midge` so hitting them causes others `Midge` to enter in a `hitaction`).

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The odds of move usage changes to be 1/2 for either move instead of being 3/5 for the aerial strike and 2/5 for the bite

## [hitaction](../../Battle%20flow/Update%20flows/Controlled%20flow.md#enemies-hitaction) support
This enemy supports `hitaction` logic and it will be performed when `hitaction` is true. The pre move logic and the aerial strike move will be done immediately after.

### Logic sequence

- [Emoticon](../../../Entities/EntityControl/EntityControl%20Methods.md#emoticon) called with the `Exclamation` emote with a time of 35
- `charge` is incremented
- `Wam` sound plays
- Yield all frames until `emoticoncooldown` expires
- Move 1 (aerial strike) is immediately used

## Move selection
2 moves are possible:

1. A single target aerial strike
2. A single target bite attack that may drain `hp`

The odds to use each move depends on if hardmode is true or note:

|Move|Odds if hardmode is false|Odds if hardmode is true|
|---:|-----|----|
|1|3/5|1/2|
|2|2/5|1/2|

However, in a `hitaction`, Move 1 is always used after the hitaction logic mentioned in the section above.

## Pre move logic
There's logic that happens before each moves, but after the `hitaction` logic if [position](../../Actors%20states/BattlePosition.md) is `Ground`:

- `checkingdead` is set to a new GotoPos coroutine on this enemy to change its [position](../../Actors%20states/BattlePosition.md) to `Flying` (`checkingdead` is set to null when the coroutine completes)
- Yield a frame
- Yield all frames until `checkingdead` is null

Here is what the coroutine effectively ends up doing:

- Over the course of 20.0 frames, `height` is lerped to `initialheight`
- `bobrange` and `bobspeed` set to `startbf` and `startbs` respectively
- `height` set to `initialheight`
- [position](../../Actors%20states/BattlePosition.md) set to `Flying`
- `checkingdead` set to null informing the caller that the coroutine is done

## Move 1 - Aerial strike
A single target aerial strike.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|1|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- Yield a frame
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near this enemy
- `PingUp` sound plays
- animstate set to 100
- Over the course of 40.0 frames, `height` increases by 2.0 via a lerp
- animstate set to 23 (`Chase`)
- `trail` set to true
- `sprite` z angle set to 20.0
- Camera moves to look near `playertargetentity`
- `PingShot` sound plays
- `FastWoosh` sound plays
- Over the course of 20.0 frames, `height` is lerped to 1.0 and position is lerped to from startp to `playertargetentity` position + (1.0, 0.0, -0.1)
- `trail` set to false
- DoDamage 1 call happens
- Yield for 0.2 seconds
- `sprite` angles reset to Vector3.zero
- Over the course of 20.0 frames, `height` is lerped to `initialheight` and position is lerped to +1.5 in x

## Move 2 - Bite attack
A single target bite attack that may drain `hp`.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|2|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- Yield a frame
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- `BugWing` sound plays on loop
- Over the course of 50.0 frames, position is lerped to `playertargetentity` position + (2.0, 0.0, -0.2)
- animstate set to 0 (`Idle`)
- Over the course of 15.0 frames, position is lerped to `playertargetentity` position + (0.5, 0.0, -0.1) and `height` is lerped to 1.15 + `playertargetentity.animid` * 0.2
- `BugWing` sound stopped
- `rotater` gets a SpriteBounce added and MessageBounce called on it which will make the sprite stretch and squeeze periodically
- animstate set to 101
- ShakeSprite called on `playertargetentity` with intensity (0.1, 0.0, 0.0) and 0.5 frametimer
- `Kiss` sound plays with 0.85 pitch and 0.9 volume
- For the next 30.0 frames, `playertargetentity` animstate is set to 11 (`Hurt`) unless `blockcooldown` hasn't expired for this frame where it's set to 24 (`Block`) instead
- Camera moves to look a bit up right
- DoDamage 1 call happens
- If `commandsuccess` is false (didn't block, ignores FRAMEONE), [Heal](../../Actors%20states/Heal.md) is called to heal this enemy by `lastdamage` amount of `hp`
- The SpriteBounce added earlier is destroyed
- Yield a frame
- `rotater` scale is set to the one before this action
- animstate set to 0 (`Idle`)
- Over the course of 20.0 frames, `height` is lerped to `initialheight` and position is lerped to be +1.5 in x
- Yield for 0.5 seconds
