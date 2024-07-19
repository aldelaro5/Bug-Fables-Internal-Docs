# `TermiteSoldier`

## Assumptions
While not required, this action assumes that this enemy is not alone in the enemy party. Typically, they will be paired with a [TermiteNasute](TermiteNasute.md), but it isn't strictly required. It mostly makes the [condition](../../Actors%20states/Conditions.md) grant move more logical.

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the projectiles throw move, the amount of hits changes to be always 1 instead of being random between 1 and 2
- In the projectiles throw move, the speed of each Projectile call changes to 27.0 from 33.0

## Move selection
3 moves are possible:

1. A single target projectiles throw that may hit multiple times
2. A single target slash attack
3. Grants 1 of 3 beneficial condition to themselves or the first enemy party member other than themselves

The decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|2/5|
|2|2/5|
|3|1/5|

However, move 3 may cause a reroll of the move selection by issuing a continue directive to the enemy action loop which restarts the entire action. The conditions in which this happens is complex and it depends who will receive the condition as well as which condition is selected to be inflicted.

The receiver is determined by doing the following:

- If this enemy is the last `enemydata` remaining, the receiver is this enemy
- Otherwise, the reciever has 6/10 chance to be this enemy and 4/10 chance to be the first enemy party member who isn't this enemy

Whoever the receiver is, the condition to inflict to them is determined randomly with the following odds:

|Condition|Odds|
|---:|---:|
|[GradualHP](../../Actors%20states/BattleCondition/GradualHP.md)|2/4|
|[AttackUp](../../Actors%20states/BattleCondition/AttackUp.md)|1/4|
|[DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md)|1/4|

After selecting both the receive and the condition, the receiver must not have the condition already for the move to be used. If they have it already, a continue directive is issued to the enemy action loop which rerolls the entire move selection process.

## Pre move logic
The following logic always happen before using a move:

- `locktri` is set to false (this doesn't do anything for this enemy)

## Move 1 - Projectiles throw
A single target projectiles throw that may hit multiple times.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen from 1 to 2 times (always 1 time if hardmode is true)|3|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|This enemy|`playertargetID`|A new sprite object rooted using the `projectilepsrites[12]` sprite (a pink pointy projectile) positioned at this enemy + (-1.65, 1.25, -0.1) with scale of 0.5x and a z angle of -90.0|33.0 (27.0 instead if hardmode is true)|0.0|null|null|null|null|Vector3.zero|false|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between thsi enemy and `playertargetentity`
- animstate set to 100
- Yield for 0.45 seconds
- The amount of hits is determined which is random between 1 and 2 (always 1 if hardmode is true). A new SpriteRenderer array with this amount to hold the projectiles
- For each hit to do:
    - `Toss3` sound plays
    - animstate set to 101
    - The projectile element is set to a new sprite object rooted using the `projectilepsrites[12]` sprite (a pink pointy projectile) positioned at this enemy + (-1.65, 1.25, -0.1) with scale of 0.5x and a z angle of -90.0
    - HitPart particles plays at the projectile element
    - Projectile 1 call happens
    - Yield for a frame
    - Yield for 0.35 seconds
    - If this is the last hit, animstate is set to 13 (`BattleIdle`)
    - Otherwise:
        - `Creak` sound plays with 1.3 volume
        - animstate set to 100
        - Yield for 0.4 seconds
- Yield all frames until all projectiles element is null (all Projectile 1 calls completed)

## Move 2 - Slash attack
A single target slash attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|4|[Flip](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (1.5, 0.0, -0.1) with a 2.0 multiplier, 1 (`Walk`) as walkstate and 102 as stopstate
- Yield all frames until `forcemove` is done
- `Creak3` sound plays
- Yield for a random interval between 0.45 and 0.65 seconds
- animstate set to 103
- `Slash2` sound plays
- DoDamage 1 call happens
- Yield for 0.5 seconds

## Move 3 - Grants a benefical condition to themselves or another enemy party member
Grants 1 of 3 beneficial condition to themselves or the first enemy party member other than themselves.

> NOTE: If the [GradualHP](../../Actors%20states/BattleCondition/GradualHP.md) condition is selected, it will be inflicted on this enemy multiple times (once for each enemy party member that exists including this enemy) which is unexpected as it was presumably supposed to inflict the condition once on each enemy party member.

See the move selection process to see how the receiver and the condition to inflict is determined.

### Logic sequence

- The receiver and the condition to inflict on them is determined as mentioned in the move selection section
- `ItemHold` sound plays
- animstate set to 4 (`ItemGet`)
- A new sprite object is created rooted at this enemy position + (-0.75, 3.0, -0.1). The sprite to use is an [item](../../../Enums%20and%20IDs/Items.md) sprite and which one is used depends on the condition to inflict:
    - [GradualHP](../../Actors%20states/BattleCondition/GradualHP.md): `CookedJellyBean`
    - [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md): `VitalitySeed`
    - [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md): `GenerousSeed`
- Yield for a second
- animstate set to the local startstate
- The item sprite is destroyed
- The logic of the condition to inflict happens:
    - [GradualHP](../../Actors%20states/BattleCondition/GradualHP.md):
        - `Heal2` sound plays
        - For each `enemydata` elements [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) is called to inflict the condition for 2 main turns on this enemy (NOT each `enemydata`) followed by the `MagicUp` particles played at this enemy position (NOT each `enemydata` position). Effectively, if there are multiple enemy party members, this enemy will get the condition for an amount of main turn equal to 2 * the amount of enemy party members
    - [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md): [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) is called to inflict the condition on the receiver for 2 main turns with effect
    - [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md): [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) is called to inflict the condition on the receiver for 2 main turns with effect
- Yield for 0.5 seconds
