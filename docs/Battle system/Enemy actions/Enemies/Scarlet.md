# `Scarlet`

## [StartBattle](../../StartBattle.md) logic
This enemy has special logic that occurs after load on StartBattle:

- `basestate` is set to 5 (`Angry`)

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: When move 5 is used (the party wide explosion move), this value is set to 4 and decreases at the end of the action including the current actor turn. Everytime that move 5 is selected, it is rerolled if this value is still higher than 0. Effectively, it's a cooldown in amount of actor turns left before being able to use the move again which is 3 full actor turns

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The [HPPercent](../../Actors%20states/HPPercent.md) threshold for getting a 999999 main turns (infinite) [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition changes to 0.65 from 0.5 until `hp` goes above this threshold at which point, the condition is removed until it reaches the threshold again. This threshold also changes the moment that the `hp` drain fraction increases from 1/2 ceiled to 2/3 ceiled for every applicable move except the party wide explosion
- On the first actor turn that [HPPercent](../../Actors%20states/HPPercent.md) threshold reaches less than 0.4, this enemy will get a 999999 mnain turns (infinite) [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition which is never removed
- In the shockwave attack move, the wave takes 25.0 frames instead of 35.0 frames to reach its target
- In the heart projectile attack move, the heart takes 60.0 frames to reach its target instead of 80.0
- In the umbrealla throw move, each half of the umbrealla trajectory takes 50.0 frames instead of 60.0 frames

## Move selection
7 moves are possible:

1. A single target slash attack that drains `hp`
2. A single target shockwave attack
3. A single target faster slash attack that inflicts [AttackDown](../../Actors%20states/BattleCondition/AttackDown.md) and drains `hp`
4. A single target heart projectile attack that inflicts [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) and drains `hp`
5. Have a flower bud appear and inflict [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) to themselves
6. A party wide explosion attack that drains `hp`
7. A single target umbrella throw that hits twice and drains `hp`

The decision of which move to use is based on odds, but the odds changes depending if [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.45 or not. Here are the odds in either situations:

|Move|[HPPercent](../../Actors%20states/HPPercent.md) >= 0.45|[HPPercent](../../Actors%20states/HPPercent.md) < 0.45|
|---:|----------------------|---------------------|
|1|2/9|3/13|
|2|2/9|1/13|
|3|1/9|3/13|
|4|1/9|Never used|
|5|1/9|Never used|
|6|1/9|3/13|
|7|1/9|3/13|

Selecting move 5 however will reroll the move selection if this enemy already has the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition.

Selecting move 6 will also reroll the move selection if [HPPercent](../../Actors%20states/HPPercent.md) is higher than 0.7 or if `data[0]` is higher than 0 (meaning the actor turns cooldown on the move hasn't yet expired, see the `data` usage section for details).

## Pre move logic
There's logic that always happen at the start of the action. There's 3 parts to it: the inifinite [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md), the infinite [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) and some logic that always happen.

### Infinite [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md)
If [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.5 (0.65 is hardmode is true) and this enemy doesn't already have the `AttackUp` condition, the following happens to inflict it (it also increases the drain fractions of all moves except the explosion attack from 1/2 to 2/3):

- `Charge10` sound plays
- animstate set to 120
- ShakeSrprite called with intensity (0.1, 0.0, 0.0) and 30.0 frametimer
- Yield for 0.5 seconds
- animstate set to 113
- `StatUp` sound plays
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 0 (red up arrow)
- Yield for 0.65 seconds
- `basestate` set to 13 (`BattleIdle`)
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the `AttackUp` condition on this enemy for 999999 main turns (infinite)

However, if they are above the `hp` threshold and they have the `AttackUp` condition with more than 10 turns, it is assumed they received the infinite condition before, but are no longer in the `hp` threshold. In that case, it is removed by doing the following (it also reverts the drain fractions of all moves except the explosion attack to 1/2):

- [RemoveCondition](../../Actors%20states/Conditions%20methods/RemoveCondition.md) called to remove the `AttackUp` condition on this enemy
- `basestate` set to 5 (`Angry`)

### Inifinite [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md)
If hardmode is true, [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.4 and this enemy doesn't already have the `DefenseUp` condition, the following happens to inflict it (it is never removed):

- `Charge10` sound plays
- animstate set to 120
- ShakeSrprite called with intensity (0.1, 0.0, 0.0) and 30.0 frametimer
- Yield for 0.5 seconds
- animstate set to 113
- `StatUp` sound plays
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 1 (blue up arrow)
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the `DefenseUp` condition on this enemy for 999999 main turns (infinite)
- Yield for 0.65 seconds

### Logic that always happen
The local startstate is always set to `basestate` (after both of the section above occured).

## Post move logic
The following logic always happen at the end of the action:

- If `data[0]` is higher than 0 (the move 5 cooldown hasn't expired yet), it is decremented

## Move 1 - Slash attack
A single target slash attack that drains `hp`.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playerdata[playertargetID]` position + (2.0, 0.0, -0.15) with a 2.0 multiplier
- Yield all frames until `forcemove` is done
- animstate set to 100
- Yield for 0.25 seconds
- `Charge10` sound plays
- ShakeSrprite called with intensity (0.1, 0.0, 0.0) and 20.0 frametimer
- Yield for 0.3 seconds
- `Gleam` particles plays with the `Gleam` sound at this enemy + (0.25, 2.1, -0.1) with 0.5 alivetime
- Yield for 0.3 seconds
- animstate set to 102
- DoDamage 1 call happens
- `HugeHit` sound plays
- ShakeScreen called with an ammount of 0.1 and 0.15 time
- Camera moves up and back a bit
- `playertargetentity` has [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on them with an h of 15.0 if `commandsuccess` is false (didn't block, ignores FRAMEONE) or 7.5 if it's true (blocked, ignores FRAMEONE)
- `playertargetentity` has SlowSpinStop called on them with a spinammount of 30.0 in y and 30.0 frametime
- Yield for 0.5 seconds
- If `lastdamage` is higher than 0, [Heal](../../Actors%20states/Heal.md) is called to heal this enemy's `hp` by that value / 2.0 ceiled. That fraction changes to be 2/3 ceiled if the first `hp` threshold is reached (see the pre move logic section above for details)
- Yield all frames until `playertargetentity` is `onground` and has its `spin`'s magnitude become 0.1 or lower

## Move 2 - Shockwave attack
A single target shockwave attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near this enemy
- animstate set to 120
- `Charge12` sound plays
- Yield for 0.25 seconds
- `CherryCharge` and `CherryBall` particles plays at this enemy position + (0.8, 1.0, -0.1) with a time of -1.0
- `CherryBall` particles has their scale zeroed out
- ShakeSrprite called with intensity (0.05, 0.0, 0.0) and 45.0 frametimer
- Over the course of 40.0 frames, `CherryBall` particles has their scale lerped to Vector3.one
- Yield for 0.25 seconds
- Camera moves to look near the midpoint between this enemy and `playertargetentity`
- `CherryCharge` and `CherryBall` particles gets destroyed
- Yield a frame
- `Lazer2` sound plays
- A new `Prefabs/Objects/WaveProjectile` object gets instantiated offscreen at -999.0 in y
- animstate set to 117
- Over the course of 35.0 frames (25.0 frames instead if hardmode is true), the `Prefabs/Objects/WaveProjectile` object moves from this enemy + (-1.0, 1.5, 0.0) to `playertargetentity` position + (0.25, 1.25, 0.0) via a lerp and its y scale gets lerped to 1.5
- `CherryBoom` particles plays at `playertargetentity` position + (0.0, 1.25, -0.1)
- `Prefabs/Objects/WaveProjectile` gets destroyed
- DoDamage 1 call happens
- Yield for 0.5 seconds

## Move 3 - Faster slash attack
A single target faster slash attack that inflicts [AttackDown](../../Actors%20states/BattleCondition/AttackDown.md) and drains `hp`.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|4|null|null|`commandsuccess`|

### Logic sequence

- Camera moves to look near this enemy
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- animstate set to 120
- `Charge12` sound plays
- Yield for 0.25 seconds
- `CherryCharge` and `CherryBall` particles plays at this enemy position + (0.8, 1.0, -0.1) with a time of -1.0
- `CherryBall` particles has their scale zeroed out
- ShakeSrprite called with intensity (0.05, 0.0, 0.0) and 45.0 frametimer
- `Woosh3` sound plays
- Over the course of 30.0 frames, `CherryBall` particles has their scale lerped to Vector3.one
- Yield for 0.25 seconds
- Camera moves a bit to the left
- `CherryCharge` and `CherryBall` particles gets destroyed
- Yield a frame
- animstate set to 101
- `trail` set to true
- Over the course of 17.0 frames, this enemy moves to `playertargetentity` position + (1.5, 0.0, -0.15) via a lerp
- `trail` set to false
- `Gleam` particles plays with the `Gleam` sound at this enemy + (0.25, 2.1, -0.1) with 0.5 alivetime
- Yield for 0.3 seconds
- animstate set to 102
- DoDamage 1 call happens
- `CherryBoom` particles plays at `playertargetentity` position + (0.0, 1.25, -0.1)
- `HugeHit` sound plays
- `StatDown` sound plays
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on `playertargetentity` with type 2 (red down arrow)
- ShakeScreen called with an ammount of 0.1 and 0.15 time
- Camera moves up and back a bit
- `playertargetentity` has [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on them with an h of 15.0 if `commandsuccess` is false (didn't block, ignores FRAMEONE) or 7.5 if it's true (blocked, ignores FRAMEONE)
- `playertargetentity` has SlowSpinStop called on them with a spinammount of 30.0 in y and 30.0 frametime
- Yield for 0.5 seconds
- If `lastdamage` is higher than 0, [Heal](../../Actors%20states/Heal.md) is called to heal this enemy's `hp` by that value / 2.0 ceiled. That fraction changes to be 2/3 ceiled if the first `hp` threshold is reached (see the pre move logic section above for details)
- Yield all frames until `playertargetentity` is `onground` and has its `spin`'s magnitude become 0.1 or lower
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called on `playertargetentity` to inflict [AttackDown](../../Actors%20states/BattleCondition/AttackDown.md) for main 2 turns (effectively 1 main turn since the current main turn gets advanced soon after)

## Move 4 - Heart projectile
A single target heart projectile attack that inflicts [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) and drains `hp`.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near this enemy
- animstate set to 104
- Yield for 0.75 seconds
- `Kiss` sound plays
- `Lazer3` sound plays
- animstate set to 105
- A new sprite object is created rooted and offscreen at 999.0 in y using the `Sprites/Particles/plainheart` sprite with a color of E57F8C (light red) with a scale of 0.65x and a ShadowLite SetUp with 0.3 opacity and 0.6 size
- Yield for 0.3 seconds
- Camera moves to look near `playertargetentity`
- Over the course of 80.0 frames (60.0 frames instead if hardmode is true), `Sprites/Particles/plainheart` moves from this enemy position + (-1.0, 1.85, -0.1) to `playertargetentity` position + (0.0, 1.0, -0.1). Before each yield, the z angle is set to Sin(Time.time * 3.0) * 10.0
- Camera moves to look near the rooted MainCam object
- `Sprites/Particles/plainheart` gets destroyed
- DoDamage 1 call happens
- `StatDown` sound plays
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on `playertargetentity` with type 3 (blue down arrow)
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called on `playertargetentity` to inflict [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) for 3 main turns (2 main turns instead if `commandsuccess` is true which means the player blocked ignoring FRAMEONE). Effectively, it's 2 or 1 main turns since the current main turns ends soon after
- Yield for 0.5 seconds
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- If `lastdamage` is higher than 0, [Heal](../../Actors%20states/Heal.md) is called to heal this enemy's `hp` by that value / 2.0 ceiled. That fraction changes to be 2/3 ceiled if the first `hp` threshold is reached (see the pre move logic section above for details)
- Yield for 0.5 seconds

## Move 5 - Flower bud giving [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md)
Have a flower bud appear and inflict [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) to themselves. No damages are dealt.

### Logic sequence

- `Disappear` sound plays
- A new `Prefabs/Objects/OpeningFlowerBud` object is created rooted at this enemy position + (0.0, 0.0, -0.1) whose SpriteRenderer color gets set to pure white half opaque
- Camera moves look near this enemy
- animstate set to 107
- Yield for 1 second
- The `Prefabs/Objects/OpeningFlowerBud`'s Animator has the `Close` animation clip played
- animstate set to 109
- `StatUp` sound plays
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 0 (red up arrow)
- `Prefabs/Objects/OpeningFlowerBud` gets destroyed
- `CherryBoom` particles plays at this enemy position + (0.0, 1.25, -0.1)
- Yield for 0.5 seconds

## Move 6 - Explosion attack
A party wide explosion attack that drains `hp`.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen, done for each player party member whose `hp` is above 0|This enemy|The player party member|3|null|{[BlockSoundOnly](../../Damage%20pipeline/DoDamage.md#blocksoundonly)}|`commandsuccess`|

### Logic sequence

- `data[0]` is set to 4 (meaning this move won't be used until 4 actor turns passes including the current one)
- Camera moves to look near this enemy
- animstate set to 111
- `Charge10` sound plays
- Yield for 0.4 seconds
- ShakeSrprite called with intensity (0.1, 0.0, 0.0) and 30.0 frametimer
- `CherryCharge` particles plays at this enemy position + (0.0, 1.5, -0.1) with a time of -1.0
- Yield for 0.5 seconds
- Camera moves to look near the player party
- Yield for 0.3 seconds
- `CherryCharge` moved offscreen at 999.0 in y
- `Charge14` sound plays
- `GrowingBuds` particles plays at this enemy + (0.15, 2.4, -0.1) with an alivetime of 0.5 and a scale of (1.5, 0.5, 2.0)
- Yield for 0.85 seconds
- `Gleam` particles plays with the `Gleam` sound at this enemy + (0.15, 2.1, -0.1) with 0.5 alivetime
- Yield for 0.5 seconds
- animstate set to 113
- `GrowingBuds` particles has LerpObject called to go -1.0 in y over 100.0 frames with destroyonend
- `CherryCharge` particles gets destroyed in 5.0 seconds
- `Explosion2` sound plays
- `FlowerImpact` particles plays at (-4.5, 0.0, 0.0)
- `impactsmoke` particles plays at (-4.5, 0.0, 0.0)
- ShakeScreen called with an ammount of 0.1 and 0.15 time
- For each player party members whose `hp` is above 0, DoDamage 1 calls happens with them
- Yield for 0.75 seconds
- If the sum of all the `lastdamage` values after each DoDamage 1 calls is higher than 0, [Heal](../../Actors%20states/Heal.md) is called to heal this enemy's `hp` by that value clamped from 0 to 8
- Yield for 0.5 seconds

## Move 7 - Umbrella throw
A single target umbrella throw that hits twice and drains `hp`.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|null|null|`commandsuccess`|
|2|Always happen after DoDamage 1|This enemy|Same as DoDamage 1|2|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between this enemy and  `playertargetentity`
- `Charge10` sound plays
- animstate set to 100
- Yield for 0.25 seconds
- ShakeSrprite called with intensity (0.1, 0.0, 0.0) and 20.0 frametimer
- Yield for 0.3 seconds
- ShakeSrprite called with intensity (0.15, 0.0, 0.0) and 30.0 frametimer
- Yield for 0.35 seconds
- animstate set to 122
- `Woosh4` sound plays
- A new sprite object is created rooted at this enemy position + (0.0, 2.0, -0.1) using the `Sprites/Entities/scarlet[18]` sprite (the umbrella) with an AudioSource setup to play the `Audio/Sounds/Woosh` audio clip on loop with a volume of MainManager.`soundvolume` * 2.0 with a maxDistance of 100.0 and a spatialBlend of 1.0
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- Over the course of 60.0 frames (50.0 instead if hardmode is true), the umbrella moves to (-15.0, 0.75, `playertargetentity`'s z position -0.1) and on the first frame that its x position becomes lower than the `playertargetentity` x position, DoDamage 1 call happen. This is immeditately followed by the reverse trajectory over the same amount of frames, but on the first frame that the umbrella's x position is higher than the `playertargetentity` x position, DoDamage 2 call happens
- The umbrella gets destroyed
- `Toss` sound plays
- `Drop` sound plays with 1.1 pitch and 0.8 volume
- animstate set to 102
- Yield for 0.5 seconds
- If the sums of DoDamage 1 and 2's `lastdamage` values is higher than 0, [Heal](../../Actors%20states/Heal.md) is called to heal this enemy's `hp` by that value / 2.0 ceiled. That fraction changes to be 2/3 ceiled if the first `hp` threshold is reached (see the pre move logic section above for details)
- Yield for 0.5 seconds
