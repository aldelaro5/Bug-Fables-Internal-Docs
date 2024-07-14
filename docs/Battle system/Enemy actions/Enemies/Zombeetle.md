# `Zombeetle`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The move usage odds changes to favor more the party wide dash move
- In the dash move, the amount of frames the entire dash takes changes to 25.0 from 33.0
- In the boulder throw move, the amount of frames the boulder takes to reach its targer changes to 41.0 from 54.0

## Move selection
2 moves are possible:

1. A party wide dash attack
2. A single target boulder throw that has a secondary hit that targets the other player party members

The decision of which move to use are based on the following odds which changes based on if hardmode is true or not:

|Move|Odds when hardmode is false|Odds when hardmode is true|
|---:|---------------------------|--------------------------|
|1|69%|59%|
|2|31%|41%|

## Move 1 - Party wide dash
A party wide dash attack.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|4|[Flip](../../Damage%20pipeline/AttackProperty.md)|`commandsuccess`|0.0|Vector3.zero|false|null|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called (but it will have no impact on this move)
- `checkingdead` is set to a new BeetleDash coroutine with the following parameters (`checkingdead` is set to null when completed):
    - enemyid: This enemy
    - damage: 4
    - animprepare: 100
    - animdash: 23 (`Chase`)
    - startp: startp
    - speed: 33 (25 instead if hardmode is true)
- Yield all frames until `checkingdead` is null

This is what the coroutine effectively ends up doing:

- `playertargetID` set to -1
- Camera moves slightly to the left
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) (-0.5, 0.0, 0.0) at 2.0 multiplier
- Yield all frames until `forcemove` is done
- `Charge3` sound plays using `sounds[9]` with 1.2 pitch
- animstate set to 100
- ShakeSprite called with (0.25, 0.0, 0.0) intensity and 60.0 frametimer
- Over the course of 41.0 frames, this enemy moves 2.0 to the left via a lerp
- ShakeSprite called with 0.1 intensity and 40.0 frametimer
- Yield for 41.0 frames counted by framestep
- `sounds[9]` stopped
- animstate set to 23 (`Chase`)
- `trail` set to true
- `FastWoosh` sound plays
- Over the course of 34 frames (26 frames instead if hardmode is true), this enemy moves to (-15.0, 0.0, 0.0) via a lerp. On the first frame that the x position is less than -4.5, the following happens only once:
    - ShakeScreen called with an ammount of (0.25, 0.0, 0.0) for 0.65 time with dontreset
    - `HugeHit` sound plays
    - PartyDamage 1 call happens
- position set to (15.0, 0.0, 0.0)
- `trail` set to false
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- animstate set to 1 (`Walk`)
- Over the course of 61.0 frames, this enemy moves to startp via a lerp while having its animstate set to 1 (`Walk`)
- `checkingdead` set to null which signals the caller that this coroutine completed

## Move 2 - Boulder throw
A single target boulder throw that has a secondary hit that targets the other player party members.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|4|null|null|`commandsuccess`|
|2|Always happen for each other player party members than DoDamage 1's target whose `hp` is above 0|This enemy|The player party member|2|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called (but it will have no impact on this move)
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) (1.5, 0.0, -0.5) with a 2.0 multiplier using 1 (`Walk`) as the walkstate and 101 as the stopstate
- Camera moves to look near `forcetarget`
- Yield all frames until `forcemove` is done
- ShakeSprite called with intensity 0.05 and 30.0 frametimer
- `Dig2` sound plays
- Yield for 0.5 seconds
- A new `Prefabs/Objects/CrackedRock` GameObject is created without its MeshCollider and Fader at this enemy position +  (-1.0, -2.0, -0.1) a scale of 0.5x and a z angle of 35.0
- animstate set to 102
- `DirtExplode` particles played at the `Prefabs/Objects/CrackedRock` position except for the y component which is 0.0
- `DigPop` sound plays with a pitch of 0.9
- ShakeScreen called with an ammount of 0.1 and a time of 0.5
- `Toss9` sound plays
- Over the course of 21.0 frames, the `Prefabs/Objects/CrackedRock` moves to + 20.0 in y via a lerp and this enemy moves to + 1.5 in x via a lerp
- animstate set to 23 (`Chase`)
- Over the course of 31.0 frames, the `Prefabs/Objects/CrackedRock` moves to 1.0 in y via a lerp. On the first frame after 22.5 frames:
    - `Toss7` sound plays
    - [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) called to move this enemy to the `Prefabs/Objects/CrackedRock` position except for the y component where it's 0.0 with a framtime of 7.0 without changeanim
- `Damage0` sound plays
- `Spin10` sound plays
- Camera moves to look near `playertargetentity` + 1.0 in y
- Over the course of 54.0 frames (41.0 frames instead if hardmode is true), the `Prefabs/Objects/CrackedRock` moves to + 1.0 in y via a BeizierCurve3 with a ymax of 7.0 and this enemy moves to + 2.0 in x via a BeizierCurve3 with a ymax of 3.0
- If there are more than 1 player party members, the ones other than `playertargetID` will be slated for getting the secondary hit. For each of them, a copy of the `Prefabs/Objects/CrackedRock` is intstantiated with halved scale
- A `Prefabs/Objects/CrackRockBreak` is instantiated with same position and scale as the `Prefabs/Objects/CrackedRock` (this gets destroyed in 60.0 seconds via the CrackRockBreak component attached to it) 
- `Prefabs/Objects/CrackedRock` is destroyed
- `RockBreak` sound plays
- DoDamage 1 call happens
- animstate set to 0 (`Idle`)
- If the secondary hit was slated to occur to the other player party members:
    - `playertargetID` set to -1
    - Over the course of ~47.95652174 frames (~36.65217391 frames instead if hardmode is true), each `Prefabs/Objects/CrackedRock` moves to their matching player party memeber position + 1.0 in y via a BeizierCurve3 with a ymax of 5.0
    - `RockBreak` sound plays with 1.1 pitch and 0.75 volume
    - For each player party members slated to receive the secondary hit:
        - Their matching `Prefabs/Objects/CrackedRock` gets destroyed
        - If their `hp` is above 0, DoDamage 2 call happens with them
- Yield for 0.5 seconds
