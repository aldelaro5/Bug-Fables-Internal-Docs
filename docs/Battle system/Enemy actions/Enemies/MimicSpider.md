# `MimicSpider`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the bubble projectiles throw move, the amount of hits to do changes to be random from 2 and 3 inclusive instead of always being 2
- In the bubble projectiles throw move, the Projectile call has 50% chance to have the [Sticky](../../Damage%20pipeline/AttackProperty.md) property instead of always being [Poison](../../Damage%20pipeline/AttackProperty.md)
- In the bubble projectiles throw move, the speed of each Projectile call changes to 30.0 from 37.0

## Move selection
2 moves are possible:

1. A single target bite attack
2. A multiple targets bubble projectiles throw

The decision of which move to use is based on the followings odds:

|Move|Odds|
|---:|----|
|1|3/5|
|2|2/5|

## Move 1 - Bite attack
A single target bite attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|[Sleep](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- `Scuttle2` sound plays via PlayMoveSound on loop using `sounds[9]` with 1.1 pitch and 0.8 volume
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (1.35, 0.0, -0.1) with 2.4 multiplier, 23 (`Chase`) walkstate and 13 (`BattleIdle`) stopstate
- Yield all frames until `forcemove` is done
- `sounds[9]` stopped
- `Chew` sound plays
- Yield for a random interval between 0.45 and 0.65 seconds
- animstate set to 107
- DoDamage 1 call happens
- Yield for 0.65 seconds
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `Scuttle2` sound plays via PlayMoveSound on loop using `sounds[9]` with 1.1 pitch and 0.8 volume
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) startp with 2.0 multiplier, 23 (`Chase`) walkstate and 13 (`BattleIdle`) stopstate
- Yield all frames until `forcemove` is done
- `sounds[9]` stopped

## Move 2 - Bubble projectiles throw
A multiple targets bubble projectiles throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen 2 times (random between 2 to 3 times inclusive instead if hardmode is true), each calls requires that at least 1 player party member is alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|2|[Poison](../../Damage%20pipeline/AttackProperty.md) (if hardmode is true, this has 50% chance to be [Sticky](../../Damage%20pipeline/AttackProperty.md) instead)|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) (target changes for each calls)|A new `Prefabs/Objects/PoisonBubble` (`Prefabs/Objects/StickyBubble` instead if the property is `Sticky`) GameObject rooted positioned at this enemy + (-1.5, 1.0, -0.1) with a scale of 0.35x|37.0 (30.0 instead if hardmode is true)|0.0|null|`PoisonEffect` (`StickyGet` instead if property is `Sticky`)|`BubbleBurst`|null|Vector3.zero|false|

### Logic sequence

- The amount of hits is determined. It's always 2 (random between 2 and 3 inclusive is hardmode is true)
- For each hits to do and as long as at least one player party member is alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)):
    - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
    - Camera moves to look near the midpoint between this enemy and `playertargetentity`
    - `Blosh` sound plays
    - animstate set to 103
    - The property of the DoDamage 1 call is determined. It's always `Poison` unless hardmode is true where it has a 50% chance to be `Sticky` instead
    - Yield for 0.75 seconds
    - `Chew` sound plays
    - `Spin10` sound plays
    - A new `Prefabs/Objects/PoisonBubble` (`Prefabs/Objects/StickyBubble` instead if the property is `Sticky`) GameObject is created rooted positioned at this enemy + (-1.5, 1.0, -0.1) with a scale of 0.35x
    - Projectile 1 call happens
    - animstate set to 104
    - Yield all frames until the bubble object is null (Projectile 1 call completed)
- Yield for 0.65 seconds
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `Scuttle2` sound plays via PlayMoveSound on loop using `sounds[9]` with 1.1 pitch and 0.8 volume
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) startp with 2.0 multiplier, 23 (`Chase`) walkstate and 13 (`BattleIdle`) stopstate
- Yield all frames until `forcemove` is done
- `sounds[9]` stopped
