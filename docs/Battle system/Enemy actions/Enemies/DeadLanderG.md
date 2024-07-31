# `DeadLanderG`

## BattleControl.[GetEXP](../../../TextAsset%20Data/Enemies%20data.md#exp-logic) special logic
This enemy is part of the set of the enemies that yields a different clamping on the rewarded amount of exp given their scaled `exp` field when they die (processed by [CheckDead](../../Battle%20flow/Action%20coroutines/CheckDead.md)).

If all of these conditions are fufilled, the rewarded amount of exp is clamped to 20 instead of 15 like most other enemies:

- [flags](../../../Flags%20arrays/flags.md) 613 is false (RUIGEE is inactive)
- `partylevel` is less than 27 (not yet at max rank)
- [Flags](../../../Flags%20arrays/flags.md) 166 is false (not during an EX mode B.O.S.S session)

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the projectiles throw move, the amount of hits changes to 4 from 3
- In the projectiles throw move, the speed of each Projectile calls changes to be random between 29.0 and 37.0 instead of being random between 34.0 and 42.0
- In the laser attack move, the second to last yield time before the DoDamage call changes to 0.75 seconds from 0.85 seconds (also changes the frametimer of the ShakeSprite call at the start of the move to be 45.0 instead of 51.0). Note that there is still a 0.1 seconds yield after that is right before the DoDamage call

## Move selection
3 moves are possible:

1. A single target claw slash attack
2. A multiple targets projectiles throw that may inflict [Sleep](../../Actors%20states/BattleCondition/Sleep.md)
3. A single target laser attack that may inflict [Sleep](../../Actors%20states/BattleCondition/Sleep.md)

The decision of which moves to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|3/7|
|2|2/7|
|3|2/7|

## Move 1 - Claw slash attack
A single target claw slash attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|4|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- `DLGammaStep` sound plays on loop using `sounds[9]`
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (3.75, 0.0, -0.15) with a 2.0 multiplier using 1 (`Walk`) as walkstate and 100 as stopstate
- Yield until `forcemove` is done
- `flip` set to false
- `sounds[9]` stopped
- `DLGammaClaw1` sound plays
- Yield for a random interval between 0.65 and 0.85 seconds
- x position decreased by -0.25
- `DLGammaClaw2` sound plays
- animstate set to 101
- DoDamage 1 call happens
- Yield for 0.5 seconds

## Move 2 - Projectiles throw
A multiple targets projectiles throw that may inflict [Sleep](../../Actors%20states/BattleCondition/Sleep.md).

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen 3 times (4 instead if hardmode is true), but each call requires that at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|2|[Sleep](../../Damage%20pipeline/AttackProperty.md)|This enemy|`playertargetID`|A new sprite object rooted using the `projectilepsrites[5]` sprite (gray spiky projectile) positioned at this enemy + (0.25, 1.25, -0.1) + a random vector from (-2.0, -0.5, 0.0) to (2.0, 0.5, 0.0) with a color of FFF8A7 (light yellow)|Random between 34.0 and 42.0 (between 29.0 and 37.0 instead if hardmode is true)|Random between 4.0 and 6.0|`keepcolor`|null|null|null|(0.0, 0.0, 15.0)|false|

### Logic sequence

- `DLGammaCharge` sound plays
- animstate set to 104
- The amount of hits is determined. It's 3 (4 instead if hardmode is true). A new SpriteRenderer array is created with this amount as the length to hold the projectiles objects
- Camera moved to look near the midpoint between this enemy and `partymiddle`
- ShakeSprite called with 0.1 intensity and 30.0 frametimer
- Yield for 0.5 seconds
- For each hit to do:
    - If there's at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)):
        - `DLGammaSpore` sound plays
        - A random number between 0.45 and 0.75 is generated
        - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
        - The projectile element is set to a new sprite object rooted using the `projectilepsrites[5]` sprite (gray spiky projectile) positioned at this enemy + (0.25, 1.25, -0.1) + a random vector from (-2.0, -0.5, 0.0) to (2.0, 0.5, 0.0)
        - DeathSmoke particles plays at the projectile element with 0.5x size
        - The projectile element's color is set to FFF8A7 (light yellow)
        - Projectile 1 call happens
        - `rotater` scale set to (1.2, 0.8, 1.0)
        - GradualScale called on `rotater` to change the scale to the value it had before this action with a frametime of 45.0 * the random number generated earlier without destroy
        - Yield for ammount of seconds equal to the random number generated earlier
- animstate set to 0 (`Idle`)
- Yield all frames until all projectiles object are null (all Projectile 1 calls completed)

## Move 3 - Laser attack
A single target laser attack that may inflict [Sleep](../../Actors%20states/BattleCondition/Sleep.md).

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|4|[Sleep](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence

- `DLGammaCharge2` sound plays
- ShakeSprite called with 0.1 intensity and 51.0 frametimer (45.0 instead if hardmode is true)
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between this enemy and `playertargetentity`
- animstate set to 102
- A new `Prefabs/Particles/Charging` GameObject is instantiated rooted at `sprite` position + (-0.8, 1.45, -0.1)
- Yield for 0.85 seconds (0.75 seconds instead if hardmode is true)
- `DLGammaLaser` sound plays
- The `Prefabs/Particles/Charging` moves offscreen at 999.0 in y then destroyed in 5.0 seconds
- animstate set to 103
- A new LightingBolt coroutine starts which will use a `Prefabs/Particles/LightingBolt` to animate a lighting using its TrailRenderer. The coroutine only contains visual logic that doesn't affect the timing of the block so its logic will be omitted
- Yield for 0.1 seconds
- DoDamage 1 call happens
- Yield for 0.5 seconds
