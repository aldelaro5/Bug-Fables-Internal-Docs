# `Thief`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the gust attack move, the [DoCommand](../../DoCommand.md) call (which is a [TappingKey](../../Action%20commands/TappingKey.md)) has `barfill` decrease 25% faster each frame. However, this change doesn't affect anything if `mashcommandalt` is true making the command a [SequentialKeys](../../Action%20commands/SequentialKeys.md)

## Move selection
3 moves are possible:

1. A single target aerial strike attack that may steal an [item](../../../Enums%20and%20IDs/Items.md)
2. A party wide wind gust attack
3. Flees the battle

Move 3 is always (and only) used if this enemy has an `holditem`.

As for the other moves, the decision of which moves to use is based on the following odds:

|Move|Odds|
|---:|----|
|1|4/10|
|2|6/10|

## Pre move logic
If the fleeing move wasn't used, the enemy will always change its [position](../../Actors%20states/BattlePosition.md) to `Flying` if it was `Ground` by doing the following:

- Over the course of 20.0 frames, `height` is lerped from 0 to `initialheight`
- `height` set to `initialheight`
- `bobrange` set to `startbf`
- `bobspeed` set to `startbs`
- [position](../../Actors%20states/BattlePosition.md) set to `Flying`

## Move 1 - Aerial strike attack with potential [item](../../../TextAsset%20Data/Items%20data.md) steal
A single target aerial strike attack that may steal an [item](../../../Enums%20and%20IDs/Items.md).

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|2|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- animstate set to 102
- Yield for 0.1 seconds
- `Gleam` particles played with `Gleam` sound at this enemy position + (0.25, 1.0 + `height`, -0.1) with an alivetime of 0.5
- Yield for 0.5 seconds
- `BugWing` sound plays via PlayMoveSound on loop using `sounds[9]`
- animstate set to 1 (`Walk`)
- Over the course of 30.0 frames:
    - `height` changes to 4.0 via a lerp
    - position is changes to (-1.0, 0.0, 0.0)
- `sounds[9]` is stopped
- `spin4` sound plays
- z `spin` set to 30.0
- ResetTrail called
- `trail` set to false
- Yield for a frame
- Yield for 0.34 seconds
- `spin` zeroed out
- `Woosh2` sound plays
- `sprite`'s angles reset to Vector3.zero
- Yield for a frame
- `trail` set to true
- Over the course of 15.0 frames, position is lerped to `playertargetentity` position
- `flip` set to true
- DoDamage 1 call happens
- If `playerdata[playertargetID]` doesn't have the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition, [StealItem](../StealItem.md) is called
- If an [item](../../../Enums%20and%20IDs/Items.md) was stolen from the call above, the local startstate is set to 4 (`ItemGet`)
- Over the course of 20.0 frames, this enemy moves to startp + the `height` before this action in y via a BeizierCurve3 with a ymax of 7.0
- position set to startp
- `height` set to the value before this action
- `trail` set to false
- `flip` set to false

## Move 2 - Wind gust attack
A party wide wind gust attack

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on any player party member whose `hp` is above 0.

### [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|165.0|[TappingKey](../../Action%20commands/TappingKey.md)|<ul><li>{4.0, 8.0, 0.8<sup>1</sup>, 1.0} if hardmode is false</li><li>{4.0, 8.0, 1.25<sup>1</sup>, 1.0} if hardmode is true</li></ul>

1: Due to the clamping logic related to this value, it means that the decrease value per frame is 0.005 * the game's frametime when hardmode is false and 0.00625 * the game's frametime when hardmode is true. Effectively, it means that hardmode being true makes `barfil` decrease 25% more per frame

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen for each player party member whose `hp` is above 0|This enemy|The player party member|1|null|null|`barfill` >= 1<sup>1</sup>|

1: The super block timing is affected by the [DoCommand timing caveat](../../Battle%20flow/GetBlock.md#known-issues-with-docommand) even if the regular block is true from suceeding DoCommand 1

### Logic sequence

- `BugWing` sound plays via PlayMoveSound on loop using `sounds[9]`
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) (-1.5, 0.0, -0.2)
- `blockcooldown` set to 0.0 which resets the blocking state to be expired (doesn't matter since there is enough yields later to make up for this)
- Camera moves to look near `partypos[0]` which is the front player party member position
- Yield all frames until `forcemove` is done
- `sounds[9]` stopped
- animstate set to 100
- [CreateHelpBox](../../Visual%20rendering/CreateHelpBox.md) called with 4 (the `TappingKey` command's help)
- `Woosh` sound plays on loop using `sounds[8]` with 0.5 pitch and 0.7 volume
- A `Prefabs/Particles/ThiefSmoke` is instantiated and positioned at (-4.0, 0.0, 0.0) rooted
- DoCommand 1 call happens
- Yield all frames until `doingaction` goes to false and that 80.0 frames passed or more. Before each frame yield:
    - All player party member whose `hp` is above 0 has their animstate set to 11 (`Hurt`) (but if `blockcooldown` is above 0 meaning a block was done, it's set to 24 (`Block`) instead)
    - All player party member whose `hp` is above 0 has their y `spin` set to 20.0 * the frame number since the start of these yields / 40.0 clamped from 0.0 to 1.0
    - `sounds[8]` pitch set to the frame number since the start of these yields / 80.0 clamped from 0.4 to 1.0
- DestroyHelpBox is called which sets `helpboxid` to -1 and destroys `helpbox` if it existed in 0.5 seconds with shrink before setting it to null
- `Prefabs/Particles/ThiefSmoke` positioned offscreen at -9999..0 in y, destroyed in 1.0 second and set to null
- `sounds[8]` stops with a 0.02 delay
- All player party member whose `hp` is above 0 has their `spin` getting zeroed out followed by DoDamage 1 call happen on them
- Yield for 0.5 seconds

## Move 3 - Flee
Flees the battle, no damages are dealt.

### Logic sequence

- `tryenemyheal` set to a new [EnemyFlee](../EnemyFlee.md) coroutine with walkstate 27 (`ItemWalk`) with afterimage on this enemy
- Yield all frames until `tryenemyheal` goes to null (the coroutine completed)
- The local fled is set to true indicating a different [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#fled-enemy-post-action) will be done
