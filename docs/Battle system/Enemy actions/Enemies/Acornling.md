# `Acornling`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does 2 changes:

- The rolling attack move now has 50% chance to end up being a headbonk attack move which can misslead the block timing. This is signaled with a slight animation changes compared to the regular rolling attack
- The rolling attack move's [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) call uses a 2.5 multiplier instead of 2.25 which makes this enemy moves ~11.111111% faster. This also applies to the headbonk attack move that is disguised as a rolling one

## Move selection
3 moves are possible:

1. A single target rolling attack
2. A single target headbonk attack
3. A single target headbonk attack disguised as a rolling attack

Move 1 and 2 have 50% each to be used.

Move 3 however can only be used when move 2 is used and hardmode is true. When these conditions are met, move 3 is done if a 50% RNG check passes.

## Move 1 - Rolling attack
A single target rolling attack

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|2|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- animstate set to 100
- Yield for 0.3 seconds
- `Jump` sound plays
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
- animstate set to 101
- Yield for 0.3 seconds
- Yield all frames until `onground` is true
- Camera moves to look near `playertargetID`
- Yield a frame
- `spin9` sounds plays on loop using `sounds[9]` with a pitch of 1.1
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (0.25, 0.0, -0.1) with a multiplier of 2.25 (2.5 instead if hardmode is true) with 101 as the walkstate and stopstate
- Yield all frames until `forcemove` is done
- `sounds[9]` stopped
- DoDamage 1 call happens
- Over the course of 40.0 frames, this enemy is moved by +2.5 in x via a BeizierCurve3 with a ymax of 4.0
- animstate set to 23 (`Chase`)
- Yield for 0.5 seconds

## Move 2 - Headbonk attack
A single target headbonk attack

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|2|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Yield a frame
- Camera moves to look near `playerdata[playertargetID]`
- animstate set to 23 (`Chase`)
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (2.0, 0.0, -0.2) with a multiplier of 2.0
- Yield all frames until `forcemove` is done
- `checkingdead` set to a new SeedlingHeadbonk coroutine with 2 damage, actionid as the attacker id, no property and with skipwalk. This coroutine sets `checkingdead` to null when completed
- Yield all frames until `checkingdead` is null

Here is what SeedlingHeadbonk effectively does with those parameters:

- `Jump` sound plays
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
- Yield for 0.3 seconds
- Yield all frames until `onground` is true
- Camera moves to look near `playertargetID`
- Yield a frame
- animstate set to 102
- `Turn` sound plays
- Over the course of 35.0 frames (but visually occurs as if it was 32.0 frames with the last 3 being padding), this enemy moves to `playertargetentity` position + (0.25, 2.8, -0.1) via a BeizierCurve3 with a ymax of 5.5 while the z angle gets a LerpAngle done to 180.0
- DoDamage 1 call happens
- Yield for 0.1 seconds
- `sprite` angles zeroed out
- animstate set to 1 (`Walk`)
- `Turn2` sound plays
- Over the course of 25.0 frames, this enemy moves to its current position + 2.0 in x with the y being 0.0 via a BeizierCurve3 with a ymax of 3.0
- `checkingdead` is set to null which signals the caller that this coroutine completed

## Move 3 - Headbonk attack disguised as rolling attack
A single target headbonk attack disguised as a rolling attack

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|2|null|null|`commandsuccess`|

### Logic sequence

The logic starts the same way than move 1, but these following directives occurs twice in a row instead of only occuring once (this gives a cue to the player about the fake rolling attack):

- `Jump` sound plays
- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
- animstate set to 101
- Yield for 0.3 seconds
- Yield all frames until `onground` is true

The [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) call also changes to have the target being a lerp between `playertargetentity` and this enemy with a factor of 0.3 (meaning 70% of the way to the target).

The rest of the logic of move 1 doesn't happen (including the DoDamage call). What happens instead is the end of what move 2 does:

- `checkingdead` set to a new SeedlingHeadbonk coroutine with 2 damage, actionid as the attacker id, no property and with skipwalk. This coroutine sets `checkingdead` to null when completed
- Yield all frames until `checkingdead` is null

As for the SeedlingHeadbonk coroutine, it does the exact same logic including the DoDamage call.
