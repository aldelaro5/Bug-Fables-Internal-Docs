# `Carmina`

## `extrastuff` usage

- `extrastuff[0]`: The `Wheel` transform used in the pre move logic which is the return of Create6Wheel
- `extrastuff[1]`: The first child of `extrastuff[0]` which is an arrow that points to an element on the wheel used in the pre logic

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the pre move logic on effect number 2 ([AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) infliction on this enemy), the amount of main turns to inflict the condition is 3 instead of 2
- In the pre move logic on effect number 3 ([DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) infliction on this enemy), the amount of main turns to inflict the condition is 3 instead of 2
- In the pre move logic on effect number 7 ([Poison](../../Actors%20states/BattleCondition/Poison.md) infliction on the player party), the amount of main turns to inflict the condition is 3 instead of 2 for each player party members (assuming they do not have the `EternalPoison` [medal](../../../Enums%20and%20IDs/Medal.md) equipped where the main turn count is always 99999 which is infinite)
- In the pre move logic on effect number 10 ([DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) infliction on the player party), the amount of main turns to inflict the condition is 3 instead of 2 for each player party members
- In the pre move logic on effect number 11 ([AttackDown](../../Actors%20states/BattleCondition/AttackDown.md) infliction on the player party), the amount of main turns to inflict the condition is 3 instead of 2 for each player party members
- In the cards throw move, the amount of cards to throw is random between 2 to 3 inclusive rather than always being 2.
- In the cards throw move, the speed of each Projectile calls changes to 22.0 from 29.0
- In the heart projectile throw move, the heart takes 60.0 frames to reach its target instead of 80.0 frames
- In the dice block hit move, the dice takes 11.0 frames to reach its target instead of 16.0 frames. NOTE: This effectively doesn't do anything in regards to blocking because the [DoDamage](../../Damage%20pipeline/DoDamage.md) call receives false as the block parameter meaning the move is unblockable

## Move selection
3 moves are possible:

1. A single target multiple cards throw
2. A single target heart projectile throw that also inflict [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md)
3. A single target dice block hit

The move usage is decided by the following odds:

|Move|Odds|
|---:|----|
|1|3/7|
|2|2/7|
|3|2/7|

## Pre move logic
There's a bunch of logic related to a random effect selection that appears on a wheel which always occur at the start of the actor turn. It's in 3 parts done in order: the wheel effect generation and setup, the application of the effect and the dimissal of the wheel.

### Wheel effect generation and setup

- animstate set to 4 (`ItemGet`)
- An array containing the numbers from 0 to 14 is created and then shuffled. The first 6 elements after the shuffle will appear on the wheel and thus are considered to be selected among the effects
- If `extrastuff` isn't null, `extrastuff[0]` (the wheel) is destroyed (this includes `extrastuff[1]` since it's childed to it)
- `extrastuff` is initialised to be 2 new Transform
- `extrastuff[0]` is initialised to the return of Create6Wheel which is a GameObject named `Wheel` representing the wheel with all elements background colors to be pure yellow with a scale of 0.35x. As for the sprites used, it depends on the first 6 elements of the shuffled effects array where it's its matching `guisprites` index that shows what the effect will do. The whole Transform is then childed to `battlemap`, scaled to 2.5x and local position set to (0.0, 25.0, 3.0)
- `extrastuff[1]` is set to the first child of `extrastuff[0]` which is the wheel's arrow pointer
- The effect to use among the 6 is generated via RNG between 0 and 5 inclusive and the corresponding element at that index will be the effect number. This means the effect is predetermined before the wheel is shown
- `extrastuff[0]` moves to (0.0, 4.0, 3.0) via MainManager.MoveTowards with 30.0 frametime with smooth and without local
- `CarminaWheel1` sound plays
- Over the course of 151.0 frames, `extrastuff[1]` z angle changes from (the effect index * 60.0 + 1500.0) to the effect index * 60.0 via a SmoothStep. Effectively, the pointer of the wheel will seemingly spin randomly, but the destination angle is already predetermined from the effect so the pointer moves according to its predetermined target element on the wheel
- `CarminaWheel2` sound plays
- Done 8 times:
    - The enablement of the sprite of the selected effect element on the wheel is toggled
    - Yield for 0.1 seconds
- The sprite of the selected effect element on the wheel is enabled
- Yield for 0.5 seconds
- animstate set to 0 (`Idle`)

### Wheel effect application
Depending on the effect number generated, the corresponding effect is applied among 15 possibilities:

- 0: [Heal](../../Actors%20states/Heal.md) is called on each player party members whose `hp` is above 0 has to heal 3 of their `hp`
- 1: [Heal](../../Actors%20states/Heal.md) is called on this enemy to heal 5 of their `hp`
- 2: [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on this enemy with effect for 2 main turns including the current one (3 main turns instead if hardmode is true)
- 3: [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition on this enemy with effect for 2 main turns (3 main turns instead if hardmode is true). The main turn count is effectively 1 (2 instead if hardmode is true) because the current main turn advances soon after this action
- 4: If `playerdata[partypointer[0]]` (the player party member in front)'s `hp` is 1 or below, `AtkFail` sound plays, otherwise:
    - [DoDamage](../../Damage%20pipeline/DoDamage.md) called to inflict an unblockable 5 damage to `playerdata[partypointer[0]]` without an attacker, property or any overrides
    - If their `hp` became 0 or lower, it is set to 1 meaning this DoDamage call can't be lethal to them
- 5:
    - [DoDamage](../../Damage%20pipeline/DoDamage.md) called to inflict an unblockable 3 damage to this enemy without an attacker, property or any overrides
    - If their `hp` became 0 or lower, it is set to 1 meaning this DoDamage call can't be lethal to them
- 6:
    - This enemy animstate is set to 11 (`Hurt`)
    - `poisoneffect` particles plays at this enemy position
    - `Poison` sound plays
    - [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [Poison](../../Actors%20states/BattleCondition/Poison.md) condition to this enemy for 3 main turns including the current one
- 7: 
    - All player party members whose `hp` is above 0 and do not have the [Sturdy](../../Actors%20states/BattleCondition/Sturdy.md) condition has the folllowing happen on them:
        - animstate set to 11 (`Hurt`)
        - [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [Poison](../../Actors%20states/BattleCondition/Poison.md) condition on the player party member with effect for 2 main turns including the current one (3 main turns instead if hardmode is true) or 99999 turns if they have the `EternalPoison` [medal](../../../Enums%20and%20IDs/Medal.md) equipped. NOTE: This means that the `poisonres` check is off by one, check the method's documentation for details
        - `poisoneffect` particles plays at the player party member position
    - `Poison` sound plays
- 8:
    - MainManager.events.MothivaSmoke particles created at this enemy position with half scale
    - `Sleep` sound plays
    - The local startstate and this enemy animstate set to 14 (`Sleep`)
    - [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [Sleep](../../Actors%20states/BattleCondition/Sleep.md) condition to this enemy for 3 main turns (The current one is normally excluded, but this effect effectively includes it by opting out of performing a move)
    - The wheel is dismissed early as outlined in the section below, but no moves will be performed on this actor turn and this action is immeditately over which simulates the [stopping](../../Actors%20states/IsStopped.md) effect of `Sleep`
- 9: 
    - All player party members whose `hp` is above 0 and do not have the [Sturdy](../../Actors%20states/BattleCondition/Sturdy.md) condition has the folllowing happen on them:
        - animstate set to 14 (`Sleep`)
        - [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [Sleep](../../Actors%20states/BattleCondition/Sleep.md) condition on the player party member with effect for 2 main turns (effectively 1 main turn since the current main turns ends soon after). NOTE: This means that the `sleepres` check is off by one, check the method's documentation for details
    - `Ping` sound plays
    - `Sleep` sound plays
    - `impactsmoke` particles plays at `partymiddle`
- 10: [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) is called for each player party member whose `hp` is above 0 to inflict the [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) condition on them with effect for 2 main turns including the current one (3 main turns instead if hardmode is true)
- 11: [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) is called for each player party member whose `hp` is above 0 to inflict the [AttackDown](../../Actors%20states/BattleCondition/AttackDown.md) condition on them with effect for 2 main turns including the current one (3 main turns instead if hardmode is true). The main turn count is effectively 1 (2 instead if hardmode is true) because the current main turn ends soon after this action
- 12 and 13: For each player party members:
    - If the player party member `hp` is above 0 and they do not have the [Sturdy](../../Actors%20states/BattleCondition/Sturdy.md) condition:
        - [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [Inked](../../Actors%20states/BattleCondition/Inked.md) condition on the player party member with effect for 2 main turns (effectively 1 main turn since the current main turns ends soon after)
        - `InkGet` particles plays at the player party member position + 1.0 in y
    - `WaterSplash2` sound plays with 0.7 pitch if it wasn't playing already
- 14:
    - This enemy's `cantmove` decrements which gives them another actor turn on this main turn (this includes this wheel logic since it's a completely new actor turn)
    - [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 5 (yellow up arrow)
    - `StatUp` sound plays

### Wheel dismisal

- Yield for 0.5 seconds
- `extrastuff[0]` (the wheel) moves to (0.0, 25.0, 3.0) via MainManager.MoveTowards with smooth and without local

## Move 1 - Cards throw
A single target multiple cards throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen 2 times (random between 2 to 3 times instead if hardmode is true)|3|Random among the following:<ul><li>null</li><li>[Sleep](../../Damage%20pipeline/AttackProperty.md)</li><li>[Poison](../../Damage%20pipeline/AttackProperty.md)</li></ul>|This enemy|`playertargetID` after [GetSingleTarger](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) (target is the same for all calls)|A new sprite object rooted with a SpriteRenderer using a `guisprite` sprite whose index depends on the property sent (90 for null which is a regular spy card, 116 for `Sleep` which is a mini boss spy card and 117 for `Poison` which is a boss spy card) positioned at this enemy + (-1.0, 1.5, -0.1) with a ShadowLite that is SetUp with 0.3 opacity and 0.5 size scaled at 0.2x and 90.0 z angle|29 (22 instead if hardmode is true)|0.0|null|null|null|null|(0.0, 0.0, -30.0)|false|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look at the midpoint between this enemy and `playertargetentity`
- A new SpriteRenderer is created to hold the cards projectile with the amount to throw which is 2 (random between 2 and 3 instead if hardmode is true)
- For each card to throw:
    - animstate set to 102
    - Yield for 0.25 seconds
    - Yield for 0.1 seconds
    - The property of the attack is determined and it's random among null, `Sleep` and `Poison`
    - `Toss` sound plays
    - The card element is set to a new sprite object rooted with a SpriteRenderer using a `guisprite` sprite whose index depends on the property sent (90 for null which is a regular spy card, 116 for `Sleep` which is a mini boss spy card and 117 for `Poison` which is a boss spy card) positioned at this enemy + (-1.0, 1.5, -0.1) with a ShadowLite that is SetUp with 0.3 opacity and 0.5 size scaled at 0.2x and 90.0 z angle
    - Projectile 1 call happens
    - animstate set to 103
    - Yield for 0.33 seconds
- Yield all frames until all the card elements in the array are null (meaning all Projectile calls completed)

## Move 2 - Heart projectile throw
A single target heart projectile throw that also inflict [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md).

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|[Numb](../../Actors%20states/BattleCondition/Numb.md)|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near this enemy
- animstate set to 104
- Yield for 0.25 seconds
- Yield for 0.1 seconds
- `Kiss` sound plays
- `Lazer3` sound plays
- A new sprite object is created rooted using the `Sprites/Particles/plainheart` sprite positioned offscreen at 999.0 in y with a color of E57F8C (light red) with a scale of 0.65x and a ShadowLite SetUp with 0.3 opacity and 0.6 size
- Yield for 0.3 seconds
- Camera moves to look near `playertargetentity`
- Over the course of 80.0 frames (60.0 frames instead if hardmode is true), the `Sprites/Particles/plainheart` moves from this enemy position + (-1.0, 1.85, -0.1) to `playertargetentity` position + (0.0, 1.0, -0.1) via a lerp. Before each frame yield, the z angle is set to Sin(Time.time * 3.0) * 10.0
- Camera moves to look near the rooted MainCam object
- `Sprites/Particles/plainheart` gets destroyed
- DoDamage 1 call happens
- `StatDown` sound plays
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on `playertargetentity` with type 3 (blue down arrow)
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called on `playertargetentity` to inflict [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) for 3 main turns (2 main turns instead if `commandsuccess` is true which means the player blocked ignoring FRAMEONE). Effectively, it's 2 or 1 main turns since the current main turns ends soon after
- Yield for 0.5 seconds

## Move 3 - Dice block hit
A single target dice block hit.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|null|`playerdata[playertargetID]` after [GetSingleTarger](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|Random between 1 and 6 inclusive|[NoExceptions](../../Damage%20pipeline/AttackProperty.md)|null|false|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- animstate set to 105
- A random number is generated between 0 and 5 inclusive which will be the number shown of the dice block and the damageammount is obtained by adding 1 to this number
- A `Prefabs/Objects/carminadice` is created rooted positioned at `playertargetentity` position + (0.0, 3.5, 0.0) with random angles between (-360.0, -360.0, -360.0) and (360.0, 360.0, 360.0)
- Camera moves to look near `playerdata[playertargetID]`
- DeathSmoke particles plats at the `Prefabs/Objects/carminadice` position
- `CarminaDice1` sound plays
- Over the course of 101.0 frames, the angles of `Prefabs/Objects/carminadice` changes from its current angles * 5.0 to one of 6 randomly chosen hardcoded values such that the number shown to the camera matches the number generated earlier + 1 via a SmoothLerp. The scale also changes from 0.25x to 0.75x via a lerp
- `Prefabs/Objects/carminadice` moves +1.0 in y via a MainManager.MoveTowards with a frametime of 15.0 with smooth and without local
- Yield for 0.25 seconds
- The MainManager.MoveTowards call is stopped
- animstate set to 106
- `CarminaDice2` sound plays
- Over the course of 16.0 frames (11.0 instead if hardmode is true), `Prefabs/Objects/carminadice` moves to `playertargetentity` position + 0.65 in y via a lerp
- `explosionsmall` particles plays at the `Prefabs/Objects/carminadice` position
- `Prefabs/Objects/carminadice` gets destroyed
- ShakeScreen called with 0.1 ammount, 0.65 time with dontreset
- DoDamage 1 call happens
- Yield for 0.5 seconds
