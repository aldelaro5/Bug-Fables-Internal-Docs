# `Centipede`

## Assumptions
It is assumed this enemy gets loaded with [SurviveWith10](../../Damage%20pipeline/AttackProperty.md) in their [weakness](../../Actors%20states/Enemy%20features.md#weakness) as it is necessary for the low `hp` phase transition move to work.

It is also assumed for the same reason that `Beetle` is present in the player party.

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 3 element with a starting value of 0.

- `data[0]`: UNUSED
- `data[1]`: A value set to 1 when gaining an [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) boost in the pre move logic and decremented (effectively set to 0) after using certain moves. This restricts the usage of the charging move where the move will not be used if this value is 1 and a reroll will occur instead. It prevents the charging move to be used on the same actor turn than gaining the `AttackUp` condition. However, it doesn't prevent the `charge` and `AttackUp` condition from stacking at the same time: when using another move than charge (which is guaranteed to happen), the value is set to 0 which makes the charging move usable on the next actor turn while the `AttackUp` condition hasn't expirred yet. Here are the moves where this value is set to 0 after their usage:
    - Draining bite attack
    - Bite attack with [Flip](../../Damage%20pipeline/AttackProperty.md)
    - Poison breath attack
    - Digging underground
- `data[2]`: A cooldown for having their [position](../../Actors%20states/BattlePosition.md) remain at `Underground`. When using the digging move, this is set to 3 and it gets decremented everytime the underground attack move is used. After this decrement, if the value reaches 0, this enemy will undig themselves setting their `position` to `Ground`. Effectively, it means this enemy can remain `Underground` for up to 3 actor turns in a row.

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The odds to gain an [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition in the pre move logic when this enemy didn't already have the condition and their [HPPercent](../../Actors%20states/HPPercent.md) is 0.65 or lower changes to 6/10 from 5/10
- In the poison breath move, the [DoCommand](../../DoCommand.md) call (which is a [TappingKey](../../Action%20commands/TappingKey.md)) has `barfill` decrease 25% faster each frame. However, this change doesn't affect anything if `mashcommandalt` is true making the command a [SequentialKeys](../../Action%20commands/SequentialKeys.md)
- In the digging underground move, `cantmove` now gets decremented at the end of the move usage which grants a free actor turn to this enemy

## Move selection
8 moves are possible:

1. A single target bite attack that can drain `hp`
2. A single target bite attack with [Flip](../../Damage%20pipeline/AttackProperty.md)
3. A party wide poison breath attack
4. Digs which sets their [position](../../Actors%20states/BattlePosition.md) to `Underground`
5. Prepares themselves to deliver a big attack by setting their `charge` to 3
6. A party wide dash attack
7. A single target underground attack that can drain `hp`
8. Initiate a low hp phase transition where the player party is killed except `Beetle`

Move 8 is always (and only) used when the following conditions are all fufilled:

- `hp` is 10 or lower
- This enemy still has [SurviveWith10](../../Damage%20pipeline/AttackProperty.md) in their [weakness](../../Actors%20states/Enemy%20features.md#weakness) (meaning this move wasn't used yet as it gets removed during the move's usage)

Move 7 is always (and only) used if move 8 wasn't and [position](../../Actors%20states/BattlePosition.md) is `Underground`.

Move 6 is always (and only) used when move 7 and 8 wasn't and `basestate` isn't 0 (`Idle`) which means move 5 wasn't used on the previous actor turn.

Move 1 through 5 can only be used when move 6 through 8 wasn't. When this is the case, the decision of which move to use among them is based on odds, but they change if `locktri` is true which means move 8 was used previously. Here are the move usage odds in either situation:

|Move|Odds when `locktri` is false|Odds when `locktri` is true|
|---:|----|-----|
|1|2/13|1/2|
|2|3/13|1/2|
|3|2/13|Never used|
|4|2/13|Never used|
|5|4/13|Never Used|

However, move 5 has these additional requirements that must be fufilled for the move to be used after it is selected:

- [HPPercent](../../Actors%20states/HPPercent.md) must be 0.6 or lower
- `data[1]` must be 0 (meaning this enemy didn't get the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on the same actor turn in the pre move logic)
- `cantmove` is 0 (meaning this enemy is doing the last actor turn they have available on this main turn)

If these requirements aren't fufilled and the move is selected, a continue directive is issued on the enemy action loop which restarts the entire action and rerolls the move selection process until another move is selected without ending the actor turn.

## Pre move logic
The following logic happens before the usage of every move except the following ones:

- Low `hp` phase transition
- Underground attack
- Party wide dash

It is composed of 3 parts that happens in order.

### [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition
This part of the pre move only happens if all of the following are fufilled:

- [HPPercent](../../Actors%20states/HPPercent.md) is 0.65 or lower
- This enemy doesn't already have the `AttackUp` condition
- A 5/10 RNG check passes (6/10 instead if hardmode is true)

When the above are fufilled, this is what the logic does:

- animstate set to 105
- `Roar2` sound plays
- ShakeScreen called with an ammount of 0.1 and 0.75 time
- Yield for 0.75 seconds
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on this enemy for 3 main turns
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 0 (red up arrow)
- `StatUp` sound plays
- `basestate` set to 100
- Yield for 0.5 seconds
- `data[1]` is set to 1 (will prevent the charging move to be used on this actor turn)

### `moves` set to 2 excluding the current main turn
This part of the pre move only happens if all of the following are fufilled:

- [HPPercent](../../Actors%20states/HPPercent.md) is 0.35 or lower
- `moves` is 1 (meaning this logic hasn't happened yet)

When the above are fufilled, this is what the logic does:

- If the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) part of the pre move didn't happen:
    - animstate set to 105
    - `Roar2` sound plays
    - ShakeScreen called with an ammount of 0.1 and 0.75 time
    - Yield for 0.75 seconds
- `moves` is set to 2 which grants 2 actor turn per main turns (excluding the current main turn)
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 5 (yellow up arrow)
- `StatUp` sound plays
- Yield for 0.5 seconds

### `basestate` logic
This part of the pre move always happen:

- `basestate` set to 0 (`Idle`)
- The local startstate is set to 0 (`Idle`)

## Move 1 - Draining bite attack
A single target bite attack that can drain `hp`.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|3|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `Step` sound plays on loop using `sounds[9]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (3.5, 0.0, -0.15) with a 2.0 multiplier
- Camera moves to look near `playertargetentity`
- Yield all frames until `forcemove` is done
- `sounds[9]` stopped
- `Bite3` sound plays
- animstate set to 100
- Yield for 0.6 seconds
- animstate set to 101
- `Bite4` sound plays
- Yield for 0.05 seconds
- DoDamage 1 call happens
- If `lastdamage` is higher than 0, [Heal](../../Actors%20states/Heal.md) called on this enemy to heal their `hp` by `lastdamage` / 2 floored clamped from 1 to `lastdamage`
- Yield for 0.65 seconds
- If `data[1]` is higher than 0 (meaning the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition was inflicted in the pre move logic), it is decremented (effectively set to 0)

## Move 2 - Bite attack with [Flip](../../Damage%20pipeline/AttackProperty.md)
A single target bite attack with [Flip](../../Damage%20pipeline/AttackProperty.md).

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|4|[Flip](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- `Step` sound plays on loop using `sounds[9]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (4.5, 0.0, -0.15) with a 2.0 multiplier
- Camera moves to look near `playertargetentity`
- Yield all frames until `forcemove` is done
- `sounds[9]` stopped
- `Bite3` sound plays
- animstate set to 102
- Yield for 0.6 seconds
- animstate set to 103
- `Woosh6` sound plays
- Yield for 0.05 seconds
- DoDamage 1 call happens
- Yield for 0.65 seconds
- If `data[1]` is higher than 0 (meaning the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition was inflicted in the pre move logic), it is decremented (effectively set to 0)

## Move 3 - Poison breath
A party wide poison breath attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|165.0|[TappingKey](../../Action%20commands/TappingKey.md)|<ul><li>{4.0, 8.0, 0.8<sup>1</sup>, 1.0} if hardmode is false</li><li>{4.0, 8.0, 1.25<sup>1</sup>, 1.0} if hardmode is true</li></ul>

1: Due to the clamping logic related to this value, it means that the decrease value per frame is 0.005 * the game's frametime when hardmode is false and 0.00625 * the game's frametime when hardmode is true. Effectively, it means that hardmode being true makes `barfil` decrease 25% more per frame

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen for each player party member whose `hp` is above 0|This enemy|The player party member|2|[Poison](../../Damage%20pipeline/AttackProperty.md)|null|true if `barfill` is 1.0 or above after DoCommand 1 (meaning the bar got filled), false otherwise|

### Logic sequence

- `Clomp` sound plays with 0.7 pitch
- animstate set to 107
- Camera moves to look at this enemy, but zoomed in
- Yield for 0.85 seconds
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- Camera moves to look near (-1.0, -0.5, 1.0)
- animstate set to 109
- `Mist` sound plays
- `poisonsmoke` particles plays near this enemy
- [CreateHelpBox](../../Visual%20rendering/CreateHelpBox.md) called with 4 (the `TappingKey` command's help)
- DoCommand 1 call happens
- All player party member whose `hp` is above 0 has their y `spin` set to 15.0
- Yield all frames until `doingaction` goes to false. Before each frame yield, all player party member whose `hp` is above 0 has their animstate set to 11 (`Hurt`) (but if `blockcooldown` is above 0 meaning a block was done, it's set to 24 (`Block`) instead)
- All player party member whose `hp` is above 0 has DoDamage 1 call happen on them followed by their `spin` getting zeroed out
- DestroyHelpBox is called which sets `helpboxid` to -1 and destroys `helpbox` if it existed in 0.5 seconds with shrink before setting it to null
- Yield for 0.5 seconds
- If `data[1]` is higher than 0 (meaning the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition was inflicted in the pre move logic), it is decremented (effectively set to 0)

## Move 4 - Digs underground
Digs which sets their [position](../../Actors%20states/BattlePosition.md) to `Underground`. No damages are dealt.

### Logic sequence

- animstate set to 110
- Yield for 0.5 seconds
- `Dig2` sound plays with 0.75 pitch
- `DirtFlying` particles plays at this enemy postion - 0.1 in x with a time of -1.0
- Yield for 0.35 seconds
- animstate set to 112
- A new sprite object is created rooted using the `Sprites/Entities/centipede` sprite at index 9 (a patch of dirt) positioned at this enemy + (-0.5, 0.0, -0.2) with a scale of (3.0, 2.0, 1.0)
- Over the course of 51.0 frames, this enemy moves from startp - 1.0 in x to starp + (-1.0, -6.0, 0.0) via a lerp
- `DirtFlying` particles moved offscreen at 999.0 in y then destroyed in 2.0 seconds
- `digging` set to true
- `digscale` set to (3.0, 1.0, 1.0)
- Yield for 0.5 seconds
- The patch of dirt is set to MainManager.Shrink over the course of 31.0 frames followed by their destruction (this is done in paralell)
- Yield for 0.5 seconds
- Yield for a frame
- animstate set to 0 (`Idle`)
- position set to startp
- `data[2]` set to 3 (this is the cooldown to remain `Underground`)
- [position](../../Actors%20states/BattlePosition.md) set to `Underground`
- If hardmode is true, `cantmove` is decremented which gives another free actor turn to this enemy
- If `data[1]` is higher than 0 (meaning the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition was inflicted in the pre move logic), it is decremented (effectively set to 0)

## Move 5 - Charging
Prepares themselves to deliver a big attack by setting their `charge` to 3. No damages are dealt.

### `dontusecharge` set to true
This move always sets `dontusecharge` to true which means `charges` will not get zeroed out in [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

### Logic sequence

- animstate set to 105
- `Roar2` sound plays with 0.8 pitch
- ShakeScreen called with an ammount of 0.1 and time of 0.75
- Yield for 0.75 seconds
- `charge` is set to 3
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on this enemy with type 4 (green up arrow)
- `StatUp` sound plays
- The local startstate and `basestate` set to 108 (this will allow the party wide dash attack move to be used on the next actor turn)
- Yield for 0.5 seconds

## Move 6 - Party wide dash
A party wide dash attack.

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|2|[Flip](../../Damage%20pipeline/AttackProperty.md)|`commandsuccess`|20.0|(0.0, 30.0, 0.0)|true|null|

### Logic sequence

- animstate set to 105
- `Roar2` sound plays
- ShakeScreen called with 0.1 ammount and 0.75 time
- Yield for 0.8 seconds
- ShakeScreen called with 0.025 ammount and -1.0 time
- `CentipedeCharge` sound plays
- animstate set to 23 (`Chase`)
- Over the course of 61.0 frames, this enemy moves to + 2.0 in x via a lerp
- ShakeSprite called with intensity of (0.1, 0.0, 0.0) and 60.0 frametimer
- Yield for 0.75 seconds
- `CentipedeCharge2` sound plays
- Over the course of 61.0 frames, this enemy moves to (-25.0, 0.0, 0.0) via a lerp. On the first frame their x position becomes lower than -4.5:
    - DoDamage 1 call happens
    - ShakeScreen called with 0.3 ammount and 0.75 time
- animstate set to 1 (`Walk`)
- Over the course of 61.0 frames, this enemy moves from (17.0, 0.0, 0.0) to startp via a lerp. Before each frame yield, their animstate is set to 1 (`Walk`)
- `basestate` and the local startstate set to 0 (`Idle`)

## Move 7 - Underground attack
A single target underground attack that can drain `hp`.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|4|[Flip](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near `playerdata[playertargetID]`
- `Digging` sound plays on loop using `sounds[9]` with 0.75 pitch
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (3.0, 0.0. -0.25) using 113 both as walkstate and stopstate
- Yield all frames until `forcemove` is done
- `sounds[9]` stopped
- animstate set to 113
- position decreased by 2.0 in y
- `digging` set to false
- `DigPop` sound plays with 0.75 pitch
- `DirtExplode` particles plays at this enemy with a y component to 0.0
- A new sprite object is created rooted using the `Sprites/Entities/centipede` sprite at index 9 (a patch of dirt) positioned at this enemy with a y component of 0.0 and -0.2 in z with a scale of (3.0, 2.0, 1.0)
- Yield for a random interval between 0.75 and 0.87 seconds
- animstate set to 114
- `Bite` sound plays
- Yield for 0.075 seconds
- DoDamage 1 call happens
- [Heal](../../Actors%20states/Heal.md) called to heal this enemy by `lastdamage` / 2 floored amount of `hp`
- Yield for 0.65 seconds
- `digging` set to true
- `Dig` sound plays
- The patch of dirt is set to MainManager.Shrink over the course of 26.0 frames followed by their destruction (this is done in paralell)
- Yield all frames until `digtime` reaches 29.0
- y position set to 0.0
- Yield for a frame
- Yield for a frame
- `data[2]` is decremented (the cooldown to have their [position](../../Actors%20states/BattlePosition.md) stay `Underground`)
- If `data[2]` just reached 0 (the cooldown expired):
    - [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
    - `Digging` sound plays oin loop using `sounds[9]` with 0.75 pitch
    - [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) startp
    - Yield all frames until `forcemove` is done
    - `sounds[9]` stopped
    - `digging` set to false
    - [position](../../Actors%20states/BattlePosition.md) set to `Ground`
    - `DigPop` sound plays with 0.75 pitch
    - `DirtExplode` particle plays at this enemy position

## Move 8 - Low `hp` phase transition
Initiate a low `hp` phase transition where the player party is killed except `Beetle`.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen to every player party members whose `hp` is above 0|This enemy|The player party member|99<sup>1</sup>|[NoExceptions](../../Damage%20pipeline/AttackProperty.md)|null|false|

1: For `Beetle` specifically, this cannot be lethal as enforced after this call

### Logic sequence

- [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) called
- Each player party member gets their [Shield](../../Actors%20states/BattleCondition/Shield.md) condition removed and their `shieldenabled` set to false
- animstate set to 105
- `Roar2` sound plays
- ShakeScreen called with 0.1 ammount and 0.75 time
- Yield for 0.75 seconds
- [RevivePlayer](../../Actors%20states/Player%20party%20members/RevivePlayer.md) called to revive `playerdata[1]` (should always be `Beetle`) leaving them at 5 `hp` without showcounter
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) called

Then, the followuing is done for each player party member in the order `Bee`, `Moth` and `Beetle`:

- If it's `Beetle`:
    - Both `Bee` and `Moth` have their `deathroutine` set to a new [Death](../../../Entities/EntityControl/Notable%20methods/Death.md) call with activatekill
    - Camera moves to look near (-2.1, 0.0, 2.5)
    - [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) Vector3.zero with 2.0 multiplier using 1 (`Walk`) as walkstate and 105 as stopstate
    - Yield all frames until `forcemove` is done
    - If `chompy` exists:
        - Their `flip` is set to false
        - They [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) to (-25.0, 0.0, 0.0) with 30.0 multiplier using 1 (`Walk`) both as walkstate and stopstate
    - [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
        - text: `|`[size](../../../SetText/Individual%20commands/size.md)`,1.5||`[halfline](../../../SetText/Individual%20commands/Halfline.md)`||`[boxstyle](../../../SetText/Individual%20commands/Boxstyle.md)`,1||`[shaky](../../../SetText/Individual%20commands/Shaky.md)`|` followed by `commondialogue[166]`
        - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
        - linebreak: `messagebreak`
        - tridimensional: false
        - position: Vector3.zero
        - size: Vector3.one
        - parent: `playerdata[1]` (`Beetle`)
        - caller: null
    - Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released
- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called with the player party member so it sets `playertargetID` and `playertargetentity` to their player party member information
- If the player party member's `hp` is above 0:
    - Camera moves to look near `playerdata[playertargetID]`
    - This enemy [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + (3.5, 0.0, -0.1) with a 4.0 multiplier using 1 (`Walk`) as walkstate and 100 as stopstate
    - Yield all frames until `forcemove` is done
    - ShakeSprite called with intensity 0.1 and 0.5 frametimer
    - Yield for 0.5 seconds
    - animstate set to 101
    - Yield for 0.05 seconds
    - ShakeScreen called with 0.15 ammount and 0.5 time
    - [RemoveCondition](../../Actors%20states/Conditions%20methods/RemoveCondition.md) called to remove the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition from the player party member and their `shieldenabled` is set to false
    - DoDamage 1 call happens
    - If it's `Beetle`, their `hp` is set to 1 which makes the damage non lethal
    - Otherwise:
        - [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) called on the player party member
        - animstate of `playerdata[1]` (`Beetle`) set to 9 (`Surprised`)
    - Yield for 0.5 seconds

After, the logic continues after the hits were done on the player party members:

- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) startp
- Yield all frames until `forcemove` is done
- `flip` set to false
- [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
    - text: `|`[shaky](../../../SetText/Individual%20commands/Shaky.md)`|` followed by `commondialogue[167]`
    - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - linebreak: `messagebreak`
    - tridimensional: false
    - position: Vector3.zero
    - size: Vector3.one
    - parent: `playerdata[1]` (`Beetle`)
    - caller: null
- Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released
- This enemy animstate set to 105
- `Roar2` sound plays
- ShakeScreen called with ammount of 0.1 and 0.75 time
- Yield for 0.75 seconds
- [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
    - text: `|`[anim](../../../SetText/Individual%20commands/Anim.md)`,this,Angry|` followed by `commondialogue[168]`
    - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - linebreak: `messagebreak`
    - tridimensional: false
    - position: Vector3.zero
    - size: Vector3.one
    - parent: `playerdata[1]` (`Beetle`)
    - caller: null
- Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released
- `Beetle` has ShakeSprite called with 0.1 intensity and 30.0 frametimer
- Yield for 0.5 seconds
- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called on `Beetle` to inflict them them [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition for 9999999 main turns (effectively infinite) with effect
- Yield for 0.5 seconds
- [flags](../../../Flags%20arrays/flags.md) 98 set to false (this disables [PepTalk](../../Player%20actions/Skills/PepTalk.md))
- `chompylock` set to true (this prevents [Chompy](../../Battle%20flow/Action%20coroutines/Chompy.md) to be called if they are present)
- `lockmmater` is set to true (this prevents the `MiracleMatter` [medal](../../../Enums%20and%20IDs/Medal.md) from taking effect)
- `Beetle` has its `lockitems` set to true which prevents them to use any [items](../../../Enums%20and%20IDs/Items.md)
- `Beetle` has its `basestate` set to 5 (`Angry`)
- [Heal](../../Actors%20states/Heal.md) called on `Beetle` to heal them to 999 `hp`
- This enemy's `charge` set to 0
- This enemy's `basestate` set to 0 (`Idle`)
- This enemy's `locktri` set to true (this is how the game tracks that this phase transition happened)
- This enemy's `moves` set to 1 (reverts them to have only 1 actor turn per main turn)
- The [SurviveWith10](../../Damage%20pipeline/AttackProperty.md) is removed from this enemy's [weakness](../../Actors%20states/Enemy%20features.md#weakness)
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [AttackDown](../../Player%20actions/Skills/AttackDown.md) condition on this enemy for 9999999 main turns (effectively infinite)
