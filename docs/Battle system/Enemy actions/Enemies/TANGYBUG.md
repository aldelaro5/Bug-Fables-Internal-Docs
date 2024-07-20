# `TANGYBUG`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the [Beetle](Beetle.md) action move logic, refer to their documentation to learn more on their hardmode changes
- In the [Mushroom](Mushroom.md) action move logic, refer to their documentation to learn more on their hardmode changes
- In the [Cape](Cape.md) action move logic, refer to their documentation to learn more on their hardmode changes
- The vine attack move may now hit twice instead of only once

## Move selection
5 moves are possible:

1. Perform the exact action logic of a [Beetle](Beetle.md)
2. Perform the exact logic of a [Mushroom](Mushroom.md)
3. Perform the exact logic of a [Cape](Cape.md)
4. Multiple seeds throws targetting random player party members
5. A single target vine attack that may hit multiple times

The decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|4/16|
|2|4/16|
|3|3/16|
|4|2/16|
|5|3/16|

## Move 1 - [Beetle](Beetle.md) action
Perform the exact action logic of a [Beetle](Beetle.md). Refer to the enemy action logic documentation to learn more.

## Move 2 - [Mushroom](Mushroom.md) action
Perform the exact action logic of a [Mushroom](Mushroom.md). Refer to the enemy action logic documentation to learn more.

## Move 3 - [Cape](Cape.md) action
Perform the exact action logic of a [Cape](Cape.md). Refer to the enemy action logic documentation to learn more.

## Move 4 - Seeds throw
Multiple seeds throws targetting random player party members.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen for half the amount of seeds thrown ceiled, but each call can only happen if [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable doesn't return -1. The amount of seeds thrown is random between 3 and 5 inclusive|2|[Poison](../../Damage%20pipeline/AttackProperty.md)|This enemy|[GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable<sup>1</sup>|A new GameObject rooted with a SpriteRenderer using the `HardSeed` [item](../../../Enums%20and%20IDs/Items.md) sprite  positioned at this enemy + 1.0 in y using the `spritemat` material on layer 0 (Default)|50.0|A random integer between 6 and 8 inclusive which is then cast to float|null|null|`WoodHit`|Empty string|(0.0, 0.0, random integer between -20 and 20 which is then cast to float)|false|

1: This targetting scheme is broken. See the [nullable GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#nullable-is-true) documentation for more details.

### Logic sequence

- `checkingdead` set to a new SeedShake coroutine called in such a way that it effectively ends up doing the following:
    - A SpriteRenderer array is created to contain the seeds with the length being the amount to throw which is random between 3 and 5 inclusive
    - For each seeds to throw:
        - The seed element of the array is initialised to a new GameObject rooted with a SpriteRenderer using the `HardSeed` [item](../../../Enums%20and%20IDs/Items.md) sprite  positioned at this enemy + 1.0 in y using the `spritemat` material on layer 0 (Default)
        - [GetRandomAvailablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) called with nullable. NOTE: This targetting scheme is broken, see the [nullable GetRandomAvailablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#nullable-is-true) documentation for details
        - If the seed index is even and the target player party member isn't -1, `Ping` sound plays followed by a Projectile 1 call. If it's odd, `PingUp` sound plays with 0.75 volume followed by the seed object moving via MainManager.ArcMovement to a random vector between (0.0, 0.0, -5.0) and (10.0, 0.0, 5.0) with destroyonend and using the same spin, height and frametime than the Projectile 1 call
        - Yield for a random interval between 0.3 and 0.6 seconds
    - Yield all frames until all elements of the seeds array are null (all Projectile 1 calls completed)
    - `checkingdead` set to null
- Yield all frames until `checkingdead` is null (SeedShake completed)

## Move 5 - Vine attack
A single target vine attack that may hit multiple times.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|`playerdata[playertargetID]`'s `hp` is above 0. Done once if hardmode is false, from 1 to 2 times if it is true as long as `playerdata[playertargetID]`'s `hp` is above 0|This enemy|The selected `playertargetID`|3 on the first hit, 2 on the second hit|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- `checkingdead` is set to a new VineAttack coroutine with 3 damageamt, actionid as the callerid with gettarget. The coroutine will set `checkingdead` to null upon completion
- Yield all frames until `checkingdead` goes to null

This is what the coroutine effectively ends up doing:

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `Rumble3` sound plays
- ShakeScreen called with an ammount of (0.1, 0.05, 0.0), time of 0.75 with dontreset
- Camera moves to look near `playertargetentity`
- The amount of hits is determined. It is 1 if hardmode is false and if it's true, it's random between 1 and 2
- An array of `Prefabs/Objects/VenusRoot` are instantiated with the amount being the amount of hits to do at (`playertargetentity` x position + a random number between -2.0 and 2.0, -5.0, `playertargetentity` z position). Each instance is set to LookAt the `playertargetentity` and has its initial position incremented by (random between -1.0 and 1.0, 0.0, -0.15)
- An array of Vector3 are instantiated with the amount being the amount of hits to do with each being the normalised direction vector from `playertargetentity` to the matching `Prefabs/Objects/VenusRoot` element
- Yield for 0.75 seconds
- For each hit:
    - `Charge8` sound plays
    - Over the course of 10.0 frames, the matching `Prefabs/Objects/VenusRoot` position lerps to its position + the matching heading direction vector * 5.0. The first time reaching the 5th frame or later when `playerdata[playertargetID]`'s `hp` is above 0, DoDamage 1 call happens before the frame yield
    - Yield for 0.5 seconds
    - If this is the last hit:
        - `Rumble3` sound plays
        - ShakeScreen called with an ammount of (0.1, 0.05, 0.0), time of 0.75 with dontreset
        - Yield for 0.5 seconds
    - The amount of damage to deal on the next hit becomes half of this one's damage floored then clamped from 1 to 99
- Over the course of 60.0 frames, all `Prefabs/Objects/VenusRoot` instances has ther scale lerped to Vector3.zero
- Yield a frame
- All `Prefabs/Objects/VenusRoot` instances gets destroyed
- `checkingdead` set to null which informs the caller that the coroutine is done
