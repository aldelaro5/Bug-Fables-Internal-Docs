# `VenusBoss`

## Assumptions
It is assumed that this enemy's [animid](../../../Enums%20and%20IDs/AnimIDs.md) (not enemy id) is `VenusGuardian`. This is because this animid has special logic in [UpdateAnimSpecific](../../../Entities/EntityControl/Animations/AnimSpecific.md#updateanimspecific) that involves `data[0]` which can influence animation logic. It also is what contains the logic of a rather complex entity structure with `extras` and `extraanims` involving animations which are bound to this animid and that the action logic assumes is present. There's also logic bound to this animid in [AnimSpecificQuirks](../../../Entities/EntityControl/Animations/AnimSpecific.md#animspecificquirks).

This enemy won't function correctly if another animid is assigned to it in [endata](../../../TextAsset%20Data/Enemies%20data.md#enemydata-data).

Additionally, it is assumed this enemy has [actimmobile](../../Actors%20states/Enemy%20features.md#actimmobile) enabled. This is necessary because if hardmode is true, it is possible for this enemy to cure some [stopping conditions](../../Actors%20states/IsStopped.md) and act instead of skipping their actor turn.

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 3 element with a starting value of 0.

- `data[0]`: A value that [UpdateAnimSpecific](../../../Entities/EntityControl/Animations/AnimSpecific.md#updateanimspecific) and [AnimSpecificQuirks](../../../Entities/EntityControl/Animations/AnimSpecific.md#animspecificquirks) uses to apply some custom animations logic. Here are the different values:
    - 0: Set when using the arm slam move, but it is effectively UNUSED because this value doesn't influence any logic
    - 1: Set when using the arm sweep move which allows to play the `ArmSweep` animation clip on `extraanims[1]` (the arm of `VenusGuardian` facing the camera) when the animstate is 102
    - 2: Set when using the grounded seeds throw move or flying in the air move which allows to play the `ArmHold` animation clip on all `extraanims` (both arms of `VenusGuardian`) when the animstate is 100 or 102
- `data[1]`: A phase tracker. There are 3 possible states:
    - 0: The initial state. Nothing special happens until [HPPercent](../../Actors%20states/HPPercent.md) is 0.65 or lower. On the first actor turn this happens, the first phase transition happens (more details in the pre logic section) and the value is set to 1
    - 1: The second state reached when [HPPercent](../../Actors%20states/HPPercent.md) reached 0.65 or lower. The value will stay at this state until [HPPercent](../../Actors%20states/HPPercent.md) is 0.375 or lower where the second phase transition happens (more details in the pre logic section) as well as using the enemy summon move for this actor turn. The value is then set to 2
    - 2: The last state. This value can no longer influence the action logic from now on when set to it
- `data[2]`: This value is UNUSED

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does 8 changes:

- If the enemy is affected by [Freeze](../../Actors%20states/BattleCondition/Freeze.md), [Sleep](../../Actors%20states/BattleCondition/Sleep.md) or [Numb](../../Actors%20states/BattleCondition/Numb.md), those conditions will be removed at the start of the actor turn so this enemy get to cure them and act immediately after instead of not performing any logic which simulates as if the actor turn was skipped
- The single target arm slam move has the yield time between the `Creak2` and `Crea6` sounds changed to be random between 0.7 and 0.9 seconds instead of being random between 0.7 and 1.05 seconds
- The party wide arm sweep move deals 1 damage per affected player party member instead of 2
- The grounded seeds throw move deals 1 damage per hit instead of 2 and the amount of seeds thrown is between 3 and 8 inclusive instead of between 3 and 7 inclusive (the amount of [Projectile](../../Damage%20pipeline/Projectile.md) calls is half of that number ceiled)
- When using the grounded seeds throw move or single target multiple hits version of the aerial seeds throw move, the chances to receive a `HardSeed` [item](../../../Enums%20and%20IDs/Items.md) in the player's standard items inventory is 75% instead of being guaranteed
- The party wide version of the aerial seeds throw move has each [DoDamage](../../Damage%20pipeline/DoDamage.md) calls deal 2 damages instead of 3. It also changes the amount of frames the seed takes to reach the player party to 22.5 frames from 30.0 frames. The odds of using this version of the move also changes to 1/3 from 1/4
- The single target multiple hits version of the aerial seeds throw move has each [Projectile](../../Damage%20pipeline/Projectile.md) calls deal 1 damages instead of 2. It also has a speed of 32.5 instead of 40.0 and the yield between throws changes to 0.45 seconds from 0.6 seconds. The odds of using this version of the move also changes to 2/3 from 3/4
- The base odds of this enemy inflicting themselves the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) or [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) conditions if they didn't already had them after using certain moves (check the post move logic section for details) changes to 36% from 21%

## [StartBattle](../../StartBattle.md) logic
After this enemy is loaded, there are some logic that happens with them on StartBattle where `extraentities` gets initialised to one element being a new entity named `Venus` with the `Venus` [animid](../../../Enums%20and%20IDs/AnimIDs.md) childed to the `battlemap` and positioned at (0.0, 0.0, 4.0). The entity also has these characteristics:

- `hologram` is set to true if [flags](../../../Flags%20arrays/flags.md) 162 is true (during a B.O.S.S or Cave Of Trials session)
- `alwaysactive` is set to true
- `activeonpause` is set to true
- A VenusBattle is added to the entity and SetUp such that it targets this enemy's `battleentity`. This allows the 2 entities to have special animation logic depending on each other's animation state. It's updated during Update, but it only includes visual changes.

## Move selection
6 moves are possible:

1. A single target arm slam attack
2. A party wide arm sweep attack
3. Multiple seeds throws targetting random player party members
4. Flies in the air to change [position](../../Actors%20states/BattlePosition.md) to `Flying`
5. Summons a new [AngryPlant](AngryPlant.md) enemy and inflicts [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) on themselves
6. An aerial seed throwing move that is either party wide or single target with multiple hits

Move 6 is only used when [position](../../Actors%20states/BattlePosition.md) is `Flying` and it is always used if that is the case. It is however possible this move is used on the same actor turn right after move 4.

Move 4 is only used and always will be used after the first phase transition, but before the second one if [position](../../Actors%20states/BattlePosition.md) is `Ground` meaning move 6 will always be used after due to the [position](../../Actors%20states/BattlePosition.md) becoming `Flying`. See the pre move logic section for more details.

Move 5 is only used once in the battle on the actor turn where the second phase transition occurs, but only if [position](../../Actors%20states/BattlePosition.md) is `Ground`. See the pre move logic section below for more details.

As for move 1, 2 or 3, they can only be used when [position](../../Actors%20states/BattlePosition.md) is `Ground` and none of the special conditions mentioned above occurs so it's basically the "standard" moveset. In that case, here are the usage odds:

|Move|Odds|
|---:|----|
|1|3/7|
|2|2/7|
|3|2/7|

## Pre move logic
There is some logic that can happen at the start of the action regardless of the situation which involves curing [stopping conditions](../../Actors%20states/IsStopped.md), some animation logic that always happen and some logic that happens only in a specific situation involving phase transitions.

### Curing [stopping conditions](../../Actors%20states/IsStopped.md)
This logic only happens if the enemy at least one of the following [stopping conditions](../../Actors%20states/IsStopped.md):

- [Freeze](../../Actors%20states/BattleCondition/Freeze.md)
- [Sleep](../../Actors%20states/BattleCondition/Sleep.md)
- [Numb](../../Actors%20states/BattleCondition/Numb.md)

What happens next depends on if hardmode is true or not. If it's false, the entire action stops here with an early break because curing stopping conditions is disabled meaning the enemy will simulate not being able to act by omiting their action completely.

If hardmode is true however, the stopping conditions gets cured allowing the enemy to act immediately after on the same actor turn. Here is the logic that happens in that case:

- `extraentities[0]` (`Venus`) animstate set to 101
- If `icecube` exists, `shakeice` is set to true
- Yield for 0.5 seconds
- `extraentities[0]` (`Venus`) animstate set to 101
- If `icecube` exists:
    - [BreakIce](../../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#breakice) called
    - `IceBreak` sound plays
- [RemoveCondition](../../Actors%20states/Conditions%20methods/RemoveCondition.md) called to remove the [Freeze](../../Actors%20states/BattleCondition/Freeze.md) condition on this enemy
- [RemoveCondition](../../Actors%20states/Conditions%20methods/RemoveCondition.md) called to remove the [Sleep](../../Actors%20states/BattleCondition/Sleep.md) condition on this enemy
- [RemoveCondition](../../Actors%20states/Conditions%20methods/RemoveCondition.md) called to remove the [Numb](../../Actors%20states/BattleCondition/Numb.md) condition on this enemy
- `isnumb` and `isasleep` set to false
- The startstate local is set to 0 (`Idle`)
- animstate set to 0 (`Idle`)
- `MagicUp` sound plays (NOTE: there is no Audioclip that exists by this name so this will silently fail)
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) called
- Yield for 0.5 seconds

### Animation logic
This always happen unless hardmode was false and the above logic caused the action to stop immediately:

- `extraentities[0]` (`Venus`)'s `anim`'s speed set to 1.0
- `extraentities[0]` (`Venus`) animstate set to 0 (`Idle`)

### Phase transition logic
There are some logic that happens before a move is selected which relates to phase transitions and first actor turn. All of the following logic are mutually exclusive: the first one mentioned that applies will be performed.

#### First actor turn
This logic happens when `data` first initialises. Since this can only happen on the first actor turn of the enemy, it can be thought of as the first actor turn logic:

- Camera moves to look near (1.25, 1.5, 2.0)
- Yield for 0.5 seconds
- [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
    - text: `commondialogue[89]`
    - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - linebreak: `messagebreak`
    - tridimensional: false
    - position: Vector3.zero
    - size: Vector3.one
    - parent: `extraentities[0]` (`Venus`)
    - caller: null
- Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md)
- Yield for 0.5 seconds

#### First phase transition
When `data[1]` transitions from 0 to 1 as [HPPercent](../../Actors%20states/HPPercent.md) reached 0.65 or lower, this is the logic that occurs:

- Camera moves to look near (1.25, 1.5, 2.0)
- Yield for 0.5 seconds
- [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
    - text: `commondialogue[90]`
    - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - linebreak: `messagebreak`
    - tridimensional: false
    - position: Vector3.zero
    - size: Vector3.one
    - parent: `extraentities[0]` (`Venus`)
    - caller: null
- Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md)
- `data[1]` is set to 1

On top of this, until the second phase transition, move 4 (flying in the air) will always be used from now on if [position](../../Actors%20states/BattlePosition.md) is `Ground` meaning move 6 (aerial seed throw) will always be used.

#### Second phase transition
When `data[1]` transitions from 1 to 2 as [HPPercent](../../Actors%20states/HPPercent.md) is 0.375 or lower, this is the logic that occurs:

- `extraentities[0]` (`Venus`) animstate set to 17 (`WeakBattleIdle`)
- Camera moves to look near (1.25, 1.5, 2.0)
- Yield for 0.5 seconds
- [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
    - text: `commondialogue[91]`
    - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - linebreak: `messagebreak`
    - tridimensional: false
    - position: Vector3.zero
    - size: Vector3.one
    - parent: `extraentities[0]` (`Venus`)
    - caller: null
- Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md)
- `data[1]` is set to 2

On top of this, move 5 ([AngryPlant](AngryPlant.md#angryplant) summon) will always be used on this actor turn, but only if [postion](../../Actors%20states/BattlePosition.md) is `Ground`. If it's `Flying`, this enemy will never use this move in the entire battle.

## Post move logic
There is some logic that may happen after a move's usage, but only if all of the following conditions are fufilled:

- Either move 1 or 2 was used or move 3 was used, but the additional quantity of seeds to throw was either 1 or 2 which has 2/3 chances to happen (2/4 instead if hardmode is true)
- Either an RNG check passes with 21% (36% if hardmode is true) or all of the following conditions are fufilled at once:
    - [HPPercent](../../Actors%20states/HPPercent.md) is 0.33 or lower
    - This enemy is the only enemy party member left
    - A 61% RNG check passes
- This enemy doesn't already have the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition
- This enemy doesn't already have the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition

If all of the above is fufilled, the enemy will inflict themselves the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) or [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition by doing the following:

- animstate and `basestate` set to 0 (`Idle`)
- `extraentities[0]` (`Venus`) animstate set to 101
- Yield for 0.65 seconds
- `extraentities[0]` (`Venus`) animstate set to 102
- `StatUp` sound plays
- The condition to inflict between [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) or [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) is decided with an RNG check. 2/3 chance to be `AttackUp` and 1/3 chance to be `DefenseUp`
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the condition determined above to this enemy for 3 main turns (effectively 2 since the main turn advances soon after)
- [StatEffect](../../Visual%20rendering/StatEffect.md) on this enemy with type 0 (red up arrow) if `AttackUp` was inflicted and type 1 (blue up arrow) if `DefenseUp` was inflicted
- Yield for 0.75 seconds

## Move 1 - Arm slash
A party wide arm slash attack

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|3|null|null|`commandsuccess`|

### Logic sequence
The arm object is reffered to `extraanims[1]`'s 6th grand children which is a bone attached to `VenusGuardian`'s arm that faces the camera.

- `extraentities[0]` (`Venus`) animstate set to 103
- `data[0]` set to 0 (doesn't end up affecting anything)
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- animstate set to 100
- `Creak2` sound plays
- Yield for a random interval between 0.7 and 1.05 seconds (the upper bound is 0.9 seconds instead if hardmode is true)
- `Creak6` sound plays
- animstate set to 102
- `extraentities[0]` (`Venus`) animstate set to 105
- `latetrans` set to the arm
- Over the course of 10.0 frames, `latepos` changes to `playertargetentity` position + 0.05 in y
- The arm has its local position set to `playertargetentity` position + 0.05 in y
- ShakeScreen called for 0.2 ammount and 0.15 time
- `Thud4` sound plays
- `impactsmoke` particles plays at `playertargetentity`
- DoDamage 1 call happens
- Yield for 0.75 seconds
- StopLate called
- The arm has its local position set to the one it had before this action

## Move 2 - Arm sweep
A party wide arm sweep attack.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls
|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen for each player party member whose `hp` is above 0|This enemy|The player party member|2 (1 instead if hardmode is true)|null|null|`commandsuccess`|

### Logic sequence
The arm object is reffered to `extraanims[1]`'s 6th grand children which is a bone attached to `VenusGuardian`'s arm that faces the camera.

- `data[0]` set to 1 (will play the `ArmSweep` animation clip on `extraanims[1]` (the arm of `VenusGuardian` facing the camera) when the animstate is set to 102)
- animstate set to 100
- `extraentities[0]` (`Venus`) animstate set to 106
- `Creak3` sound plays
- Over the course of 30.0 frames, `sprite`'s angles lerped to (0.0, -30.0, 0.0)
- ShakeSprite called with (0.1, 0.1, 0.1) intensity and 0.5 frametimer
- Yield for 0.5 seconds
- animstate set to 102
- `extraentities[0]` (`Venus`) animstate set to 108
- `Woosh6` sound plays with 0.75 pitch
- Over the course of 40.0 frames, `sprite` y angle gets LerpAngle to 25.0. When reaching the 20th frame or later for the first time:
    - `impactsmoke` particles played at `partymiddle`
    - `Dust` sound plays
    - DoDamage 1 call happens for every affected player party members
- Yield for 0.5 seconds
- The arm has its local postion set to the one it had before this action

## Move 3 - Grounded seeds throw
Multiple seeds throws targetting random player party members.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen for half the amount of seeds thrown ceiled, but each call can only happen if [GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable doesn't return -1. The amount of seeds thrown is random between 3 and 7 inclusive (between 3 and 8 inclusive instead if hardmode is true)|2 (1 if hardmode is true)|null|This enemy|[GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) with nullable<sup>1</sup>|A new GameObject rooted with a SpriteRenderer using the `HardSeed` [item](../../../Enums%20and%20IDs/Items.md) sprite  positioned at this enemy + (0.0, 3.0, 0.5) using the `spritemat` material on layer 0 (Default)|60.0 (50.0 instead if hardmode is true)|A random integer between 6 and 8 inclusive which is then cast to float|null|null|`WoodHit`|Empty string|(0.0, 0.0, random integer between -20 and 20 which is then cast to float)|false|

1: This targetting scheme is broken. See the [nullable GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#nullable-is-true) documentation for more details.

### Logic sequence

- The amount of additional throws is generated to be between 1 and 3 (between 1 and 4 instead if hardmode is true). This will be added later to the base amount which is between 2 and 4 (but the additional amout influences the post move logic, see its section for details)
- `data[0]` set to 2 which will play the `ArmHold` animation clip on all `extraanims` (both arms of `VenusGuardian`) when the animstate is 100 or 102
- `animstate` set to 100
- `Creak2` sound plays
- `extraentities[0]` (`Venus`) animstate set to 101
- Yield for 0.625 seconds
- animstate set to 102
- `extraentities[0]` (`Venus`) animstate set to 102
- Yield for 0.25 seconds
- animstate set to 104
- ShakeScreen called for 0.15 ammount and -1.0 time
- `Thud3` sound plays
- Yield for 0.5 seconds
- MainManager.`screenshake` set to (0.09, 0.09, 0.09)
- The amount of seeds to throw is determined. It's random between 3 and 7 inclusive (between 3 and 8 inclusive instead if hardmode is true)
- `checkingdead` set to a new SeedShake coroutine called in such a way that it effectively ends up doing the following (it receives the amount of seeds to throw):
    - A SpriteRenderer array is created to contain the seeds with the length being the amount to throw
    - For each seeds to throw:
        - The seed element of the array is initialised to a new GameObject rooted with a SpriteRenderer using the `HardSeed` [item](../../../Enums%20and%20IDs/Items.md) sprite  positioned at this enemy + (0.0, 3.0, 0.5) using the `spritemat` material on layer 0 (Default)
        - [GetRandomAvailablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) called with nullable. NOTE: This targetting scheme is broken, see the [nullable GetRandomAvailablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#nullable-is-true) documentation for details
        - If the seed index is even and the target player party member isn't -1, `Ping` sound plays followed by a Projectile 1 call. If it's odd, `PingUp` sound plays with 0.75 volume followed by the seed object moving via MainManager.ArcMovement to a random vector between (0.0, 0.0, -5.0) and (10.0, 0.0, 5.0) with destroyonend and using the same spin, height and frametime than the Projectile 1 call
        - Yield for a random interval between 0.3 and 0.6 seconds
    - Yield all frames until all elements of the seeds array are null (all Projectile 1 calls completed)
    - `checkingdead` set to null
- Yield all frames until `checkingdead` is null (SeedShake completed)
- animstate set to 0 (`Idle`)
- Over the course of 20.0 frames, MainManager.`screenshake` gets lerped to Vector3.zero
- `extraentities[0]` (`Venus`) animstate set to 8 (`Happy`)
- MainManager.`screenshake` set to Vector3.zero
- [GetRandomAvailablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) called with nullable. NOTE: This targetting scheme is broken, see the [nullable GetRandomAvailablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#nullable-is-true) documentation for details
- `checkingdead` set to a new HardSeedVenus coroutine call with the target player party member index just obtained as a and the hardmode value as harmode which effectively does the following:
    - The corutine does nothing except setting `checkingdead` to null if not all of the following conditions are fufilled:
        - `hasblocked` is true (At least one Projectile call had its DoDamage call blocked)
        - The target isn't -1
        - `items[0]` is less than instance.`maxitems` (not full standard [items](../../../Enums%20and%20IDs/Items.md) inventory)
        - Either hardmode is false or a 75% RNG check passes if it is true
    - If the above are all fufilled, some animation logic occurs and `items[0]` gets a `HardSeed` [item](../../../Enums%20and%20IDs/Items.md) appended to it 
- Yield all frames until `checkingdead` is null (HardSeedVenus completed)

## Move 4 - Flies in the air
Flies in the air to change [position](../../Actors%20states/BattlePosition.md) to `Flying`. No damages are dealt, but since move 6 (aerial seeds throw) is always used after this move, damages will be dealt there.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which doesn't affect anything for this move.

### Logic sequence

- `extraentities[0]` (`Venus`) animstate set to 101
- Camera moves to look to the enemy party zoomed out and angled downward
- `data[0]` set to 2 which will play the `ArmHold` animation clip on all `extraanims` (both arms of `VenusGuardian`) when the animstate is 100 or 102
- animstate set to 104
- `bobrange` and `bobspeed` reset to `startbf` and `startbs` respectively
- Yield for 0.75 seconds
- The first 2 `extras` (`VenusGuardian`'s wings) are enabled and has the following happen on each:
    - scale set to Vector3.zero
    - x angle set to 150.0 for the first wing and -150.0 for the second wing
    - `LeafExplode` sound plays
    - `leafexplode` particles plays at the wing with scale (3.25, 3.25, 1.5)
- Over the course of 15.0 frames, both wings has their scale lerped to their initial value before they were set to Vector3.zero
- animstate set to 100
- Yield for 0.75 seconds
- `extraentities[0]` (`Venus`) animstate set to 102
- Yield for 0.5 seconds
- `Flap` sound plays on loop
- Over the course of 60.0 frames, `height` is lerped to 4.0 and when it goes higher than 0.25, animstate is set to 0 (`Idle`) before each frame yield
- `Flap` sound stopped
- animstate set to 0 (`Idle`)
- `height` set to 4.0
- [position](../../Actors%20states/BattlePosition.md) set to `Flying`
- `basestate` and the startstate local set to 13 (`BattleIdle`)
- Yield for 0.5 seconds

## Move 5 - Enemy summon
Summons a new [AngryPlant](AngryPlant.md) enemy and inflicts [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) on themselves. No damages are dealt.

### Logic sequence

- `extraentities[0]` (`Venus`) animstate set to 103
- Yield for 0.5 seconds
- `extraentities[0]` (`Venus`) animstate set to 105
- `Charge` sound plays
- `checkingdead` set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon an [AngryPlant](AngryPlant.md) with type being `FromGround` at (6.0, 0.0, -1.0) without cantmove (meaning they will be able to act immediately after this action)
- Yield all frames until `checkingdead` is null (the coroutine completed)
- Yield for 0.5 seconds
- `extraentities[0]` (`Venus`) animstate set to 101
- Yield for 0.5 seconds
- `extraentities[0]` (`Venus`) animstate set to 102
- `StatUp` sound plays
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) on this enemy for 3 main turns (effectively 2 since this is always the last actor turn of this enemy)
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 1 (blue up arrow)
- Yield for 0.75 seconds

## Move 6 - Aerial seeds throw
An aerial seed throwing move that is either party wide or single target with multiple hits.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Only happens in the party wide version of the move and done for each player party member whose `hp` is above 0|This enemy|The player party member|3 (2 instead if hardmode is true)|null|null|`commandsuccess`|

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Only happens in the single target multiple hits version of the move from 2 to 3 times|2 (1 instead if hardmode is true)|null|This enemy|`playertargetID`|A new GameObject rooted with a SpriteRenderer using the `HardSeed` [item](../../../Enums%20and%20IDs/Items.md) sprite  positioned at this enemy + (0.0, 3.0, 0.5) using the `spritemat` material|40.0 (32.5 instead if hardmode is true)|0.0|null|null|`WoodHit`|Empty string|(0.0, 0.0, 20.0)|false|

### Logic sequence
The version of the move to use is decided first with an RNG check. The odds are 3/4 (2/3 instead if hardmode is true) for the single target multiple hits version and 1/4 (1/3 instead if hardmode is true) for the party wide version.

The flower SpriteRenderer is `extra[3]` which is the flower at the end of `VenusGuardian`'s arm facing the camera.

- Camera moves a bit up left
- `extraentities[0]` (`Venus`) animstate set to 103
- If using the single target version:
    - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
    - `Jump2` sound plays
- If using the party wide version:
    - The flower's color is set to pure red
    - `Charge3` sound plays with 1.25 pitch
- animstate set to 106
- Yield for an interval that depends on the version of the move:
    - Single target: random between 0.6 and 0.8 seconds
    - Party wide: 1.25 seconds
- The amount of seeds to throw is determined:
    - Single target: random between 2 and 3
    - Party wide: 1
- A SpriteRenderer array is created to contain each seeds to throw
- For each seeds to throw:
    - `extraentities[0]` (`Venus`) animstate set to 105
    - animstate set to 108
    - The seed array initialised to a new GameObject rooted with a SpriteRenderer using the `HardSeed` [item](../../../Enums%20and%20IDs/Items.md) sprite  positioned at this enemy + (0.0, 3.0, 0.5) using the `spritemat` material
    - The flower color changes to pure white
    - If this is the party wide version of the move:
        - `PingShot` sound plays with 0.6 pitch
        - HitPart particles plays near the flower object
        - Over the course of 40.0 frames (32.5 frames instead if hardmode is true), the seed position is lerped to `partymiddle` and before each frame yield, it Rotate in z by 30.0 * the game's framtime
        - `explosion` particles plays at `partymiddle`
        - `Explosion` sound plays
        - ShakeScreen with an ammount of (0.2, 0.2, 0.2) and time of 0.25
        - DoDamage 1 call happens for each affected player party members
        - The seed is destroyed
    - If this is the single target version of the move:
        - Projectile call 1 happens
        - `PingShot` sound plays
    - Yield for 0.5 seconds
    - If this is the last throw:
        - `extraentities[0]` (`Venus`) animstate set to 103
        - animstate set to 106
        - `Jump2` sound plays
        - Yield for 0.6 seconds (0.45 seconds instead if hardmode is true)
- `extraentities[0]` (`Venus`) animstate set to 8 (`Happy`)
- Yield all frames until all the seeds elements are null (meaning that all throws are done no matter the version of the move used)
- Yield for 0.5 seconds
- animstate set to 0 (`Idle`)

After, if the single target version of the move was used:

- [GetRandomAvailablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) called with nullable. NOTE: This targetting scheme is broken, see the [nullable GetRandomAvailablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#nullable-is-true) documentation for details
- `checkingdead` set to a new HardSeedVenus coroutine call with the target player party member index just obtained as a and the hardmode value as harmode which effectively does the following:
    - The corutine does nothing except setting `checkingdead` to null if not all of the following conditions are fufilled:
        - `hasblocked` is true (At least one Projectile call had its DoDamage call blocked)
        - The target isn't -1
        - `items[0]` is less than instance.`maxitems` (not full standard [items](../../../Enums%20and%20IDs/Items.md) inventory)
        - Either hardmode is false or a 75% RNG check passes if it is true
    - If the above are all fufilled, some animation logic occurs and `items[0]` gets a `HardSeed` [item](../../../Enums%20and%20IDs/Items.md) appended to it 
- Yield all frames until `checkingdead` is null (HardSeedVenus completed)
