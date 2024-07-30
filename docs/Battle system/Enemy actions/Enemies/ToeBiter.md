# `ToeBiter`

## BattleControl.[GetEXP](../../../TextAsset%20Data/Enemies%20data.md#exp-logic) special logic
This enemy is part of the set of the enemies that yields a different clamping on the rewarded amount of exp given their scaled `exp` field when they die (processed by [CheckDead](../../Battle%20flow/Action%20coroutines/CheckDead.md)).

If all of these conditions are fufilled, the rewarded amount of exp is clamped to 20 instead of 15 like most other enemies:

- [flags](../Flags%20arrays/flags.md) 613 is false (RUIGEE is inactive)
- `partylevel` is less than 27 (not yet at max rank)
- [Flags](../Flags%20arrays/flags.md) 166 is false (not during an EX mode B.O.S.S session)

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: This is a cooldown for the behavior the rock pickup move has where it may or may not issue a continue directive to the enemy action loop which would inevitably cause them to use the rock throw move on the same actor turn instead of waiting on the next actor turn. It only has an effect if hardmode is true as this behavior never happens if it is false regardless of this value. When hardmode is true, it is set to 3 on the first time the rock pickup move is used. Everytime the rock throw move is used, the value is decremented. For the rock to be thrown on the same actor turn than its pickup, the value needs to be 0 or lower. Effectively, it's an antispam on hardmode where this enemy will need to wait 2 other rock throws usage before being able to throw the next one on the same actor turn than its pickup

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the rock pickup move, it is now possible to use the rock throw move immediately on the same actor turn if `data[0]` is 0 or less (see the `data` section for details on how this cooldown works)
- In the rock throw move, the amount of frames the rock takes to reach its target changes to 36.0 frames from 43.5 frames

## Move selection
3 moves are possible:

1. Picks up a rock in their `helditem` for throwing it later. This may lead to the usage of move 3 immediately on the same actor turn
2. A single target arms slam attack
3. A single target rock throw from the one held in their `helditem`

Move 3 is always (and only) used when `helditem` isn't null.

For the other moves, the decision of which move to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|1/2|
|2|1/2|

However, move 1 is special because after picking up the rock, it is possible to issue a continue directive to the enemy action loop which restarts the entire action on the same actor turn. Since `helditem` won't be null on the second iteration, it will inevitably lead to the usage of move 3 still on the same actor turn than picking up the rock. In order for this to happen, the following conditions must be fufilled by the end of move 1:

- hardmode is true
- `data[0]` is 0 or less (see the `data` section for more details on how this cooldown works)

If the above conditions aren't fufilled by the end of move 1, the action ends as usual.

## Move 1 - Rock pickup
Picks up a rock in their `helditem` for throwing it later. This may lead to the usage of the rock throw move immediately on the same actor turn. No damages are dealt by this move, but if the rock throw move is used immediately after, that move will deal damages.

### Logic sequence

- A new `Prefabs/Objects/CrackedRock` GameObject is created without its MeshCollider and Fader positioned offscreen at (0.0, -999.0, 0.0) with a scale of 0.65x
- animstate set to 100
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- `RockPluck` sound plays
- ShakeScreen called with 0.25 ammount and 0.2 time
- `DirtExplode2` particles plays at this enemy position + (-2.0, 0.0, -0.1)
- animstate set to 101
- Over the course of 11.0 frames, `Prefabs/Objects/CrackedRock` moves from this enemy position + (-2.0, -3.0, -0.1) to this enemy position + its `itemoffset` via a lerp
- CreateHeldItem is called with -2 as the itemid which sets `holditem` to -2 and initialises this enemy's `helditem` to be a new SpriteRenderer with no sprite set childed to this enemy's `sprite` using the `spritemat` material on layer 14 (`Sprite`) with a FaceCamera component
- `Prefabs/Objects/CrackedRock` gets childed to this enemy's `helditem`
- animstate and the local startstate set to 4 (`ItemGet`)
- If hardmode is true and `data[0]` is 0 or less (the cooldown expired, see the `data` section for more details), a continue directive is issued to the enemy action loop which restarts the entire action on the same actor turn. This will lead to the immediate usage of the rock throw move because `helditem` got initialised

## Move 2 - Arms slam
A single target arms slam attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|3|[Pierce](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|null|`commandsuccess`|

1: Enemy piercing damages are disabled so this property does nothing, see the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md#piercing) documentation to learn more

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (2.25, 0.0, -0.1) with 2.0 multiplier using 1 (`Walk`) as walkstate and 102 as stopstate
- Yield all frames until `forcemove` is done
- `Creak` sound plays with 1.2 pitch
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- ShakeScreen called with 0.15 ammount and 0.5 time with dontreset
- `Toss16` sound plays
- animstate set to 28 (`TossItem`)
- DoDamage 1 call happens
- Yield for 0.5 seconds

## Move 3 - Rock throw
A single target rock throw from the one held in their `helditem`.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|5|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between this enemy and `playertargetentity`
- `Charge19` sound plays
- ShakeSprite called with 0.2 intensity and 45.0 frametimer
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- animstate set to 28 (`TossItem`)
- `Toss12` sound plays
- Over the course of 43.5 frames (36.0 frames instead if hardmode is true), this enemy's `helditem` moves to `playertargetentity` position + (0.0, 2.0. -0.1)
- ShakeScreen called with 0.2 ammount and 0.65 time with dontreset
- DoDamage 1 call happens
- `RockBreak` sound plays
- A `Prefabs/Objects/CrackRockBreak` is instantiated childed to this enemy's `helditem` with same position and scale as the `Prefabs/Objects/CrackedRock` (this gets destroyed in 60.0 seconds via the CrackRockBreak component attached to it) 
- This enemy's `helditem` is destroyed (eventually will be set to null which prevents to use this move on the next actor turn at first)
- `holditem` set to -1
- animstate and the local startstate set to 0 (`Idle`)
- `data[0]` is decremented
- Yield for 0.5 seconds
