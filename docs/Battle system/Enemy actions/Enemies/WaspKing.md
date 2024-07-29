# `WaspKing`

## Assumptions
It is assumed that this enemy is loaded with [actimmobile](../../Actors%20states/Enemy%20features.md#actimmobile) set to true because it is required for this enemy to be able to cure themselves of [stopping condition](../../Actors%20states/IsStopped.md) if hardmode is true.

It is also assumed this enemy's [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath) is set to its [matching EventDialogue](../../Battle%20flow/EventDialogues/WaspKing.md).

## [Fire](../../Actors%20states/BattleCondition/Fire.md) damage infliction logic in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md)
This enemy has a 30% chance to be on fire on a `Fire` property attack.

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: An actor turn cooldown on the usage of the fire pillar move. When the move is used, the value is set to 3. In the post move logic, the value is decremented if it is above 0. For the fire pillar move to be usable, the value needs to be above 0. Effectively, it's an antispam for the fire pillar move where this enemy needs to wait 2 full actor turns before the move becomes usable again

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- If this enemy has a [stopping condition](../../Actors%20states/IsStopped.md), it will now be cured in the pre move logic by calling [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) on this enemy
- In the axe throw move, the amount of frames the axe takes to reach its target changes to 21.0 frames from 30.0 frames
- In the fire balls projectiles move, the odds to use the single target version of the move when [HPPercent](../../Actors%20states/HPPercent.md) is lower than 0.7 changes to 40% from 30%
- In the fire balls projectiles move (single target version), the amount of frames the fire ball takes to reach its target changes to 26.0 frames from 34.0 frames
- In the fire balls projectiles move, (multiple targets version), the speed of each Projectile call changes to be random between 22.0 and 28.0 instead of being random between 29.0 and 35.0

## Move selection
5 moves are possible:

1. A single target axe slash attack
2. A single target axe throw attack
3. A party wide fire pillar attack
4. A single or multiple targets fire balls projectiles throw
5. Heals themselves for some `hp` and gets either the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) or [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition

The decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|3/12|
|2|2/12|
|3|2/12|
|4|3/12|
|5|2/12|

However, if move 3 is selected and `data[0]` isn't 0 (the cooldown on this move's usage hasn't expired yet), a continue directive is issued to the enemy action loop which restarts the entire action and rerolls the move selection process.

Also, the same happens if move 5 was selected and any of the following are true:

- [HPPercent](../../Actors%20states/HPPercent.md) is above 0.8
- `moves` is above 1 (meaning it was upgraded to 2 in the phase transition logic of the pre move logic) while `cantmove` is higher than -1 (meaning this enemy is doing their last actor turn of the main turn)
- This enemy already has the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition
- This enemy already has the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition

## Pre move logic
The following logic always happen before using a move. It is in 2 parts done in order: stop conditions curing and phase transition.

### Stop condition curing
This part only happens if hardmode is true and this enemy [IsStopped](../../Actors%20states/IsStopped.md) with skipimmobile:

- ShakeScreen called with an ammount of 0.1 and 0.5 time with dontreset
- `Charge7` sound plays with 0.8 pitch
- ShakeSprite called with 0.1 intensity and 30.0 frametimer
- Yield for 0.5 seconds
- [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) called on this enemy
- `Heal3` sound plays
- Yield for 0.5 seconds

### Phase transition
This part only happens if [HPPercent](../../Actors%20states/HPPercent.md) is 0.4 or lower and `locktri` is false (meaning this part hasn't happened before as it will be set to true) where a phase transition happens. Notably, `moves` is set to 2 with `cantmove` decremented (this enemy will now get 2 actor turns per main turn including the current main turn) as well as increases the damageammount of the axe slash move by 1:

- `Charge7` sound plays with 0.8 pitch
- animstate set to 1 (`Walk`)
- ShakeSprite called with 0.1 intensity and 30.0 frametimer
- Yield for 0.5 seconds
- `cantmove` gets decremented and `moves` set to 2 which gives this enemy 2 actor turns per main turn including the current main turn
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 5 (yellow up arrow)
- `StatUp` sound plays
- Yield for 0.5 seconds
- `locktri` set to false so this logic isn't done again, but it also increases the damageammount of the axe slash move by 1

## Post move logic
The following logic always happen after using a move:

- If `data[0]` is above 0, it is decremented

## Move 1 - Axe slash attack
A single target axe slash attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|5 (6 instead if `locktri` is true which means the phase transition in the pre move logic occured before)|[Flip](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

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

## Move 2 - Axe throw attack
A single target axe throw attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|6|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- animstate set to 100
- Yield for 0.65 seconds
- `WaspKingAxToss` sound plays
- animstate set to 114
- A new sprite object is created childed to the `battlemap` using the `Sprites/Entities/waspking` index 62 sprite (their axe) positioned at this enemy position + (-1.0, 2.0, 0.0)
- `WaspKingAxSpin` sound plays
- Over the course of 30.0 frames (21.0 frames instead if hardmode is true), the axe moves to `playertargetentity` position + (0.0, 1.25, -0.1) via a lerp. Before each frame yield, the axe's z angle increases by 52.0 * MainManager.`framestep`
- `WaspKingAxSpin` sound plays
- ShakeScreen called with 0.2 ammount and 0.5 time
- DoDamage 1 call happens
- Over the course of 46.0 frames, the axe moves to this enemy position + (-1.0, 2.0, 0.0) via a BeizierCurve3 with a ymax of 5.0. Before each frame yield, the axe's z angle increases by 25.0 * MainManager.`framestep`
- The axe object gets destroyed
- `WaspKingAxeThrowCatch` sound plays
- animstate set to 101
- Yield for 0.75 seconds
- animstate set to 103
- Yield for 0.35 seconds

## Move 3 - Fire pillar attack
A party wide fire pillar attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|4|[Fire](../../Damage%20pipeline/AttackProperty.md)|`commandsuccess`|0.0|Vector3.zero|false|null|

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
- `data[0]` is set to 3
- `checkingdead` is set to null which signals the caller that this coroutine completed

## Move 4 - Fire balls projectiles
A single or multiple targets fire balls projectiles throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Only happens on the single target version of the move|This enemy|The selected `playertargetID`|5|[Fire](../../Damage%20pipeline/AttackProperty.md)|{[BlockSoundOnly](../../Damage%20pipeline/DoDamage.md#blocksoundonly)}|`commandsuccess`|

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Only happens on the multiple targets version of the move from 2 to 3 times (always 3 times instead if `moves` isn't 1 meaning the phase transition in the pre move logic occured before), but each calls requires that there's at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|2|[Fire](../../Damage%20pipeline/AttackProperty.md)|This enemy|`playertargetID`|A new `Prefabs/Particles/Fireball` GameObject childed to the `battlemap` positioned at this enemy + (-1.25, 2.5, 0.0) with a scale of 0.75x|Random between 29.0 and 35.0 (random between 22.0 and 28.0 instead if hardmode is true)|2.0|`SepPart@2@4`|`Fire`|`WaspKingMFireball2`|null|Vector3.zero|false|

### Logic sequence

The version of the move is determined first. It is the single target version ifthere's at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))
- [HPPercent](../../Actors%20states/HPPercent.md) is less than 0.7
- A 30% RNG check passes (40% instead if hardmode is true)

The multiple targets version is used otherwise.

#### Single target version

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `WaspKingFireball1` sound plays
- animstate set to 112
- Yield for 0.25 seconds
- A `Prefabs/Particles/Fireball` is instantiated childed to the `battlemap` positioned at this enemy position + (-1.25, 2.5, 0.0)
- Over the course of 31.0 frames, the `Prefabs/Particles/Fireball`'s scale changes from 0.0x to 1.0x via a lerp
- animstate set to 115
- Over the course of 61.0 frames, the `Prefabs/Particles/Fireball` moves to the same position via a BeizierCurve3 with a ymax of 15.0. Before each frame yield on the second half, animstate is set to 100
- animstate set to 101
- `WaspKingFireball2` sound plays
- ShakeScreen called with 0.1 ammount and 0.4 time with dontreset
- Over the course of 34.0 frames (26.0 frames instead if hardmode is true), the `Prefabs/Particles/Fireball` moves to `playertargetentity` position + 0.9 in y via a BeizierCurve3 with a ymax of 3.0
- ShakeScreen called with 0.2 ammount and 0.5 time with dontreset
- `Fire` particles plays at the `Prefabs/Particles/Fireball` position
- `BigHit` sound plays
- DoDamage 1 call happens
- `Prefabs/Particles/Fireball` is moved offscreen at -9999.0 in y then destroyed in 4.0 seconds
- Yield for 0.5 seconds

#### Multiple targets version

- The amount of hits is determined. It's random from 2 to 3 inclusive, but if `moves` isn't 1, it's always 3 (meaning the phase transition in the pre move occured before). A new Transform array is created with this amount as the length to hold the projectiles objects
- For each hits to do, if there's at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)):
    - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
    - `WaspKingMFireball1` sound plays
    - animstate set to 112
    - Yield for 0.25 seconds
    - The projectile element is set to a new `Prefabs/Particles/Fireball` GameObject childed to the `battlemap` positioned at this enemy + (-1.25, 2.5, 0.0)
    - Over the course of 21.0 frames, `Prefabs/Particles/Fireball`'s scale changes from 0.0x to 0.75x via a lerp
    - Yield for 0.1 seconds
    - animstate set to 113
    - Projectile 1 call happens
    - Yield for a random interval between 0.35 and 0.6 seconds
- Yield all frames until all projectiles objects are null (all Projectile 1 calls completed)

## Move 5 - Heals `hp` with [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) or [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition
Heals themselves for some `hp` and gets either the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) or [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition. No damages are dealt.

### Logic sequence

- animstate set to 4 (`ItemGet`)
- Yield for 0.25 seconds
- `ItemHold` sound plays
- The effect is determined among the following determined with uniform odds:
    - Heal 5 `hp` with [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition using the `BurlyChips` [item](../../../Enums%20and%20IDs/Items.md) sprite
    - Heal 4 `hp` with [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition using the `FrenchFries` [item](../../../Enums%20and%20IDs/Items.md) sprite
- A new sprite object is created childed to `battlemap` using the item sprite of the effect positioned at this enemy position + (-0.85, 2.65, -0.1). The object is destroyed in 0.75 seconds
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the condition of the effect to this enemy for 3 main turns (effectively 2 since the current main turn ends soon enough) with effect
- Yield for 0.5 seconds
- [Heal](../../Actors%20states/Heal.md) called to heal this enemy by the amount of `hp` the effect heals by
- Yield for 0.5 seconds
