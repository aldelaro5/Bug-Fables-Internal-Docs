# `LeafbugClubber`

## Assumptions
It is assumed this enemy is loaded with a non empty [chargeonotherenemy](../../Actors%20states/Enemy%20features.md#chargeonotherenemy) as it is needed for their `hitaction` logic to work. Typically, it will be set to other `LeafbugClubber`, [LeafbugArcher](LeafbugArcher.md) and [LeafbugNinja](LeafbugNinja.md).

## [hitaction](../../Battle%20flow/Update%20flows/Controlled%20flow.md#enemies-hitaction) support
This enemy supports `hitaction` logic and it will be performed when `hitaction` is true instead of any moves.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- If `charge` is less than 3, it is incremented with the following:
    - animstate set to 5 (`Angry`)
    - `Wam` sound plays
    - [Emoticon](../../../Entities/EntityControl/EntityControl%20Methods.md#emoticon) called on this enemy with the `Exclamation` emote with 20 time
    - Yield all frames until `emoticoncooldown` expires
    - `StatUp` sound plays
    - [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 4 (green up arrow)
    - Yield for 0.5 seconds
    - `charge` is incremented

## Move selection
2 moves are possible:

1. A single target club slash
2. A party wide club jumping attack

The decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|3/4|
|2|1/4|

## Pre move logic
The following logic is always done at the start of the action if this enemy is the last `enemydata` remaining:

- `Charge10` sound plays with 0.9 pitch
- ShakeSprite called with intensity of (0.1, 0.0, 0.0) and 25.0 frametimer
- animstate set to 5 (`Angry`)
- Yield for 0.3 seconds
- `StatUp` sound plays
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 0 (red up arrow)
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on this enemy for 1 main turn (only the current action)
- Yield for 0.5 seconds

## Move 1 - Club slash
A single target club slash.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|4|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (1.5, 0.0, -0.1) with a 2.0 multiplier
- Yield all frames until `forcemove` is done
- `Rope2` sound plays with 1.1 pitch
- animstate set to 100
- Yield for 0.65 seconds
- animstate set to 102
- `Rope2` sound plays
- `Death3` sound plays
- DeathSmoke particles plays at `playertargetentity`
- ShakeScreen called with an ammount of 0.2 and 0.5 time
- DoDamage 1 call happens
- Yield for 0.75 seconds

## Move 2 - Club jumping attack
A party wide club jumping attack.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|3|null|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence

- animstate set to 5 (`Angry`)
- `checkingdead` set to a new JumpPartyAttackCoroutine (`checkingdead` is set to null when completed)
- Yield all frames until `checkingdead` is null

This is what the coroutine ends up doing:

- ShakeSprite called with intensity of (0.1, 0.0, 0.0) and 30.0 frametimer
- `Grow1` sound plays with 0.8 pitch
- Yield for 0.5 seconds
- Camera moves to look near `partymiddle`
- animstate set to 103
- `Jump` sound plays
- `Launch` sound plays
- Over the course of 41.0 frames, this enemy moves from its position with a y component of 0.0 to (-4.0, 0.0, 0.0) via a BeizierCurve3 with a ymax of 6.0
- animstate set to 102
- `Launch` sound plays
- `Explosion5` sound plays
- ShakeScreen called with an ammount of 0.2 and 0.5 time
- `impactsmoke` particles plays at this enemy position - 0.5 in x
- PartyDamage 1 call happens
- Yield for 0.75 seconds
- `checkingdead` set to null which signals the caller that this coroutine completed
