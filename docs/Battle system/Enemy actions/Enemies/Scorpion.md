# `Scorpion`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the bould throw move, the time the boulder takes to reach its target changes to 46.0 frames from 61.0 frames

## Move selection
3 moves are possible:

1. A single target claw swipe that can hit twice
2. A single target tail thrust that may inflict [Sleep](../../Actors%20states/BattleCondition/Sleep.md)
3. A party wide boulder throw

The decision of which move to use is determined by the following odds:

|Move|Odds|
|1|3/7|
|2|2/7|
|3|2/7|

## Move 1 - Claw swipes
A single target claw swipe that can hit twice.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|3|null|null|`commandsuccess`|
|2|Only happens if after DoDamage 1, `hp` / `maxhp` floored is 0.5 or less (50% or less `hp` remaining)|This enemy|The same `playertargetID` as DoDamage 1|3|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (3.0, 0.0, -0.15) with 2.0 multiplier
- Yield all frames until `forcemove` is done
- animstate set to 100
- `ScorpionClaw1` sound plays
- Yield for 0.75 seconds
- animstate set to 102
- `ScorpionClaw2` sound plays
- Yield for 0.05 seconds
- DoDamage 1 call happens
- If `hp` / `maxhp` is 0.5 or less (50% or less `hp` remaining):
    - Yield for 0.35 seconds
    - `ScorpionClaw1` sound plays with 0.95 pitch
    - animstate set to 104
    - Yield for 0.5 seconds
    - `ScorpionClaw2` sound plays
    - animstate set to 106
    - Yield for 0.05 seconds
    - DoDamage 2 call happens

## Move 2 - Tail thrust
A single target tail thrust that may inflict [Sleep](../../Actors%20states/BattleCondition/Sleep.md).

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|3|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### About incorrect [Poison](../../Damage%20pipeline/AttackProperty.md) and [Sleep](../../Actors%20states/BattleCondition/Sleep.md) properties logic
While the DoDamage call features a `Pierce` property, the move ends up doing an incorrect version of the `Poison` or `Sleep` property logic due to the [single property shortcoming](../../Damage%20pipeline/Known%20design%20issues.md#it-is-impossible-to-call-dodamage-using-multiple-attackproperty). Here are the differences:

- On FRAMEONE, regular blocking will incorrectly prevent the condition from being inflicted which doesn't happen normally
- When inflicting `Poison`, the `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) won't work despite the attack being physical
- When inflicting `Sleep`, the amount of turns to inflict is 2 instead of 3 even if the target didn't already had the condition
- When inflicting `Poison` and the target had the `Sleep` condition without a `HeavySleeper` [medal](../../../Enums%20and%20IDs/Medal.md), its `Sleep` condition is not removed when it otherwise would have been. NOTE: in practice, nothing changes because the DoDamage 1 call will remove the `Sleep` condition anyway
- When inflicting `Sleep`, there is no resistance increase
- The condition infliction can still happen if the target has the [Numb](../../Actors%20states/BattleCondition/Numb.md) while having the `ShockTrooper` [medal](../../../Enums%20and%20IDs/Medal.md) equipped
- When inflicting `Poison`, the `Poison` sound is not played when it should have been
- When inflicting `Sleep`, the `Sleep` sound is not played when it should have been
- When inflicting `Sleep`, `isasleep` is not set to true when it should have been. NOTE: this doesn't affect anything since the value gets fixed when the main turn advance occurs and no potential impact can happen under normal gameplay until it advances

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (3.5, 0.0, -0.15) with 2.0 multiplier
- Yield all frames until `forcemove` is done
- animstate set to 107
- `ScorpionTail3` sound plays
- Yield for a random amount of time between 0.65 seconds and 0.85 seconds
- animstate set to 109
- `ScorpionTail4` sound plays
- Yield for 0.05 seconds
- DoDamage 1 call happens
- If `commandsuccess` is false (didn't blocked, ignores FRAMEONE) and `playerdata[playertargetID]` doesn't have the [Shield](../../Actors%20states/BattleCondition/Shield.md) or [Sturdy](../../Player%20actions/Skills/Sturdy.md) conditions, [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) is called on `playerdata[playertargetID]` to inflict a randomly chosen condition between [Poison](../../Actors%20states/BattleCondition/Poison.md) and [Sleep](../../Actors%20states/BattleCondition/Sleep.md) for 2 main turns (effectively 1 main turn since the current one is advanced soon). NOTE: This ignores status resistance
- Yield for 0.75 seconds

## Move 3 - Boulder throw
A party wide boulder throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|4|null|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence

- animstate set to 110
- `Dig2` sound plays
- `Digging` particles plays at this enemy position -3.0 in x
- A new `Prefabs/Objects/CrackedRock` GameObject is created without its MeshCollider and Fader with a position of (0.0, -10.0, this enemy z position + 0.3) a scale of 3x and an x angle of 180.0
- Yield for 0.65 seconds
- The `Digging` particles are moved offscreen at -999.0 in y
- animstate set to 111
- Yield for 0.35 seconds
- `RockPluck` sound plays
- animstate set to 112
- ShakeScreen called with an ammount of 0.25 and a time of 0.2
- `DirtExplode2` particles plays at this enemy position -3.0 in x
- Over the course of 11.0 frames, the `Prefabs/Objects/CrackedRock` moves from this enemy postion + (-3.5, -1.25, -0.1) to this enemy position + (-0.85, 5.45, 0.0) via a BeizierCurve3 with a ymax of 2.0
- Yield for 0.3 seconds
- animstate set to 114
- Yield for 0.1 seconds
- `Toss7` sound plays
- Over the course of 61.0 frames (46.0 instead if hardmode is true), the `Prefabs/Objects/CrackedRock` moves to (-4.5, 1.0, 0.0) via a BeizierCurve3 with a ymax of 6.0
- `Thud5` sound plays
- `impactsmoke` particles plays at the `Prefabs/Objects/CrackedRock` position
- A `Prefabs/Objects/CrackRockBreak` is instantiated with same position and scale as the `Prefabs/Objects/CrackedRock` (this gets destroyed in 60.0 seconds via the CrackRockBreak component attached to it) 
- `Prefabs/Objects/CrackedRock` is destroyed
- PartyDamage 1 call happens
- Yield for 0.5 seconds
