# `MakiTutorial`

> NOTE: This enemy action is very complex because it plays an important part in the [basic combat tutorial](../../Battle%20flow/Combat%20tutorials.md#basic-combat-tutorial). It is recommended to check the documentation about it alongside this page to understand everything

## Asumptions
It is assumed that fighting this enemy means that the basic combat tutorial was not given yet and that the player party is composed of only `Bee` and `Beetle` in default party order. 

It is also assumed that [flags](../../../Flags%20arrays/flags.md) 15 is false (haven't given the basic combat tutorial yet) and that [flagvar](../../../Flags%20arrays/flagvar.md) is either 1 or 3 every time this action logic runs.

## Move selection
1 move is possible:

1. A single target sword slash attack

Move 1 is always done.

## Move 1 - sword slash attack
A single target sword slash attack

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`<sup>1</sup>|2|null|null|`commandsuccess`<sup>1</sup>|

1: In the case of the basic combat tutorial, the tartget is `playerdata[0]` (should always be `Bee`) and the block value is true which guarantees a regular block, but not necessarily a super block. More info available at the [block tutorial documentation](../../Battle%20flow/Combat%20tutorials.md#block-tutorial)

### Logic sequence

- If `flagvar` 11 is 1
    - The [turn flow tutorial](../../Battle%20flow/Combat%20tutorials.md#turn-flow-tutorial) logic occurs
- Otherwise
    - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) is called
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) the `plauyertargetID` player party member + (2.0, -0.15 * `globalcamdir.forward`, 0.0) with a multipler of 1.3333334
- Yield all frames until `forcemove` is done
- animstate set to 101
- Yield for 0.25 seconds
- If `flagvar` 11 is 1
    - The [block tutorial](../../Battle%20flow/Combat%20tutorials.md#block-tutorial) logic occurs
- Otherwise
    - Yield for 0.2 seconds
- `FastWoosh` sound plays
- animstate set to 103
- DoDamage 1 call happens
- Yield for 0.5 seconds
- If `flagvar` 11 is less than 3 (not all basic combat tutorials were given yet), it is set to 2 (which advances the tutorial to the next phase)
- Yield all frames until `inevent` is false (shouldn't be true in general because the last [EventDialogue](../../Battle%20flow/EventDialogue.md) involved here is long finished)
