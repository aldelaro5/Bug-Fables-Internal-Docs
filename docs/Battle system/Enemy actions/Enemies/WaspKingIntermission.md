# `WaspKingIntermission`

## Assumptions
This enemy is a lite version of [WaspKing](WaspKing.md), they actually share logic, but this enemy imposes heavy restrictions. They will be taken for granted in this page for the sake of simplicity.

It is assumed that MainManager.`battleloseevent` is true because this enemy will deal an absurdly high damaging party wide attack on the 3rd main turn which is meant to kill the player party. Setting this field to true allows [DeadParty](../../Battle%20flow/Terminal%20coroutines/DeadParty.md) to not call [GameOver](../../Battle%20flow/Terminal%20coroutines/GameOver.md) and simply exit the battle via [ReturnToOverworld](../../Battle%20flow/Terminal%20coroutines/ReturnToOverworld.md).

It is assumed that this enemy is immune to [Freeze](../../Actors%20states/BattleCondition/Freeze.md), [Sleep](../../Actors%20states/BattleCondition/Sleep.md) and [Numb](../../Actors%20states/BattleCondition/Numb.md) conditions which avoids delaying the usage of his fire pillar move since this enemy can't be [stopped](../../Actors%20states/IsStopped.md).

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: Effectively UNUSED here because while it normally would be an actor turn cooldown to using the fire pillar move, this enemy has it disabled so it has no effect. It is set to 3 when the move is used and decremented after, but this will be ignored and have no effects

## Move selection
2 moves are possible:

1. A single target axe slash attack
2. A party wide fire pillar attack that always kills the player party

Move 1 is always used until `turns` is 2 or above where move 2 is always used (meaning on the 3rd main turn, this enemy is locked to use move 2).

## Pre move logic
The following logic is always done at the start of the action:

- `hp` is set to 999 (effectively making this enemy impossible to kill as their data should also be 999 `maxhp`)
- If `turns` is 2 or above (2 main turns fully advanced since the start of the battle):
    - [SetText](../../../SetText/SetText.md) is called in [dialogue](../../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
        - text: `commondialogue[119]`
        - [fonttype](../../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
        - linebreak: `messagebreak`
        - tridimensional: false
        - position: Vector3.zero
        - size: Vector3.one
        - parent: This enemy
        - caller: null
    - Yield all frames until the [message](../../../SetText/Notable%20states.md#message) lock is released
    - Every player party members with a [Shield](../../Actors%20states/BattleCondition/Shield.md) condition has it removed via [RemoveCondition](../../Actors%20states/Conditions%20methods/RemoveCondition.md)

## Move 1 - Axe slash
A single target axe slash attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|5|[Flip](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: This property gets overriden to null in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) as the target is a player party member so it does nothing.

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (2.0, 0.0, -0.1) with a 2.0 multiplier
- Yield all frames until `forcemove` is done
- animstate set to 100
- Yield for 0.65 seconds
- ShakeScreen called with 0.2 ammount and 0.5 time
- animstate set to 101
- `Slash` sound plays
- DoDamage 1 call happens
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on `playertargetentity`
- Yield for 0.75 seconds
- animstate set to 103
- Yield for 0.35 seconds
- Yield all frames until `playertargetentity` is `onground`

## Move 2 - Fire pillar instakill
A party wide fire pillar attack that always kills the player party.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|99|[NoExceptions](../../Damage%20pipeline/AttackProperty.md)|false|0.0|Vector3.zero|false|null|

### Logic sequence

- animstate set to 105
- `checkingdead` set to a new FirePillar coroutine from this enemy without green
- Yield all frames until `checkingdead` is null (the coroutine completed)

Here is what the coroutine ends up doing:

- Yield for 0.5 seconds
- `FirePillar` sound plays
- A new `Prefabs/Objects/FirePillar 1` GameObject is instantiated childed to the `battlemap` at `partymiddle` with a scale of (0.0, 1.0, 0.0 )and a DialogueAnim with the following:
    - `targetscale`: (0.75, 1.0, 0.75)
    - `shrink`: false
    - `shrinkspeed`: 0.015
- Yield for 0.65 seconds
- `Prefabs/Objects/FirePillar 1`'s DialogueAnim's `shrinkspeed` set to 0.2
- Yield for 0.2 seconds
- ShakeScreen called with ammount of 0.25 and 0.75 time
- PartyDamage 1 call happens
- Yield for 1.15 seconds
- `Prefabs/Objects/FirePillar 1`'s DialogueAnim's `targetscale` set to (0.0, 1.0, 0.0)
- All ParticleSystem under `Prefabs/Objects/FirePillar 1` are stopped
- `Prefabs/Objects/FirePillar 1`'s DialogueAnim's `shrinkspeed` set to 0.05
- `Prefabs/Objects/FirePillar 1` gets destroyed
- `data[0]` is set to 3 (effectively does nothing because this `data` slot is unused for this enemy)
- `checkingdead` is set to null which signals the caller that this coroutine completed
