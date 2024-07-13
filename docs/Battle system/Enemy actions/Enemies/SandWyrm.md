# `SandWyrm`

## Special enemy party layout
This enemy is meant to function in tandem with another: the [SandWyrmTail](SandWyrmTail.md).

The `SandWyrmTail` should always be present when this enemy is not [stopped](../../Actors%20states/IsStopped.md) and the battle should always starts with both. This is enforced by the [EnemyCheck](../../StartBattle%20phases/Pre%20haltbattleload.md#enemycheck) logic and this enemy's pre move action logic. They also have each a VenusBattle that references each other so they can have synchronized and consistent animstate logic.

In this configuration, the `SandWyrmTail` is more an instrument to this enemy even if they are both completely separate enemies. The action logic of either assumes the presence of the other with the only exception being when the `SandWyrmTail` dies where its [EventDialogue](../../Battle%20flow/EventDialogues/SandWyrm%20and%20SandWyrmTail.md) will inflict [Flipped](../../Actors%20states/BattleCondition/Flipped.md) to `SandWyrm` for 2 main turns. This effectively will make them not act for 1 main turn due to how DoAction will skip the action if more than 1 main turns are left on it (but on the next main turn, the condition gets removed and they get to act again so overall, it's 1 main turn of no action).

At that moment, the `SandWyrm` is stopped for a main turn and the `SandWyrmTail` removed since they died. However, the moment the `SandWyrm` gets to act again on the next main turn, they will summon the `SandWyrmTail` again. Essentially, killing the tail will only remove it temporarilly from the battle.

This setup is tore down on `SandWyrm`'s EventDialogue when it dies. In that case, both it and the `SandWyrmTail` will be killed together if the `SandWyrmTail` still exists by then (it is possible it didn't since it's possible it died and `SandWyrm` couldn't summon it).

## Assumptions
It is assumed that this enemy has an `eventondeath` set to the proper [EventDialogue](../../Battle%20flow/EventDialogues/SandWyrm%20and%20SandWyrmTail.md#sandwyrm-and-sandwyrmtail-eventdialogues) because this EventDialogue will allow the [SandWyrmTail](SandWyrmTail.md) to die with this enemy.

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: The value determines the enemy to summon in the enemy summon move and it toggles between 0 and 1 after the summon each time the move is used. A value of 0 (the starting value) means a [Strider](Strider.md) will be summoned while a value of 1 means a [DivingSpider](DivingSpider.md) will be summoned

## [StartBattle](../../StartBattle.md) special logic
This enemy features special logic during StartBattle before and after it is loaded.

### Before the enemy is loaded ([EnemyCheck](../../StartBattle%20phases/Pre%20haltbattleload.md#enemycheck))
If StartBattle was called with any elements of enemyids containing this enemy or [SandWyrmTail](SandWyrmTail.md), the entire enemyids is overriden to be {`SandWyrm`, [SandWyrmTail](SandWyrmTail.md)}.

### After the enemy is loaded ([enemy party setup](../../StartBattle%20phases/Post%20haltbattleload.md#enemy-party-setup))
If `enemydata` contains this enemy, a VenusBattle is added to the first 2 elements of the enemy party where the parent is the other enemy and the target is themselves. Since the EnemyCheck logic already enforced the layout of the enemy party, it effectively means that:

- This enemy gets a VenusBattle SetUp with a target of themselves and the parent being the [SandWyrmTail](SandWyrmTail.md)
- The [SandWyrmTail](SandWyrmTail.md) gets a VenusBattle SetUp with a target of themselves and the parent being this enemy

This means that each have a VenusBattle that checks each other to make sure they have matching animstate and animations. It allows to keep them in sync visually in a consistent manner.

This part of the logic also changes the `battlepos` of this enemy and the [SandWyrmTail](SandWyrmTail.md):

- This enemy: (3.0, 0.0, 0.0)
- The [SandWyrmTail](SandWyrmTail.md): (5.9, 0.0, 0.0)

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the bubble spray move, the [DoCommand](../../DoCommand.md) call (which is a [TappingKey](../../Action%20commands/TappingKey.md)) has `barfill` decrease 25% faster each frame. However, this change doesn't affect anything if `mashcommandalt` is true making the command a [SequentialKeys](../../Action%20commands/SequentialKeys.md)

## Move selection
3 moves are possible:

1. A single target bite attack
2. A party wide bubble spray attack
3. Summons a new [Strider](Strider.md) or [DivingSpider](DivingSpider.md) enemy

The decision of the move to use is based on odds, but those odds changes if InputIO.IsEditor is true which can't happen under normal gameplay (it would mean the game is being ran inside the Unity editor). Here are the odds:

|Move|Odds in Unity player|Odds in Unity editor|
|1|3/6|3/7|
|2|2/6|4/7|
|3|1/6|Never used|

However, if move 3 is selected and there's already 3 or more enemy party members, a continue directive is issued on the enemy actions loop which restart the entire action logic and effectively rerolls the move selection.

## Pre move logic
There's logic that will always be done at the start of the action. It's in 2 parts done in order.

### [SandWyrmTail](SandWyrmTail.md) summon
If `SandWyrmTail` isn't in `enemydata`, it is summoned back by doing the following (normally happens when this enemy acts after the tail died):

- ShakeScreen called with an ammount of 0.05 and a time of 1.35
- animstate set to 100
- `checkingdead` set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon a new [SandWyrmTail](SandWyrmTail.md) enemy with type `Raise` at (5.9, 0.0, 0.0)
- Yield for a frame
- The `SandWyrmTail` has their `height` and `initialheight` set to 0.0
- Yield for 1.35 seconds
- `Prefabs/Objects/WaterSplash` instiantiated at (5.9, 0.0, 0.0) and destroyed in 0.75 seconds
- Yield all frames until `checkingdead` is null (the summoning completed)
- `size` set to 1.75
- `cursoroffset` set to (-1.75, 4.45, 0.0)
- The local startstate and `basestate` set to 0 (`Idle`)
- Yield for 0.5 seconds

### Changing startp
The following logic always happen if there's exactly 2 enemy party members (this essentially restores the startp this enemy had before it summoned an enemy):

- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) (3.0, 0.0, 0.0)
- Yield all frames until `forcemove` is done
- startp is set to this enemy position except the y component which is set to 0.0
- `flip` is set to false

## Move 1 - Bite attack
A single target bite attack.

### [BasicAttack](../../Damage%20pipeline/BasicAttack.md) calls

|#|Conditions|enemyid|walkstate|attackstate|attackstate2|damage|offset|property|shake|delay|sounds|dontgettarget|
|-:|---------|-------|--------|------------|------------|------|-----|--------|-----|-----|------|-------------|
|1|Always happen|This enemy|1 (`Walk`)|100|101|3|(3.25, 0.0, -0.1)|null|0.0|Random between 0.65 seconds and 0.85 seconds|`TidalWormBite`|false|

### Logic sequence

- `checkingdead` is set to the BasicAttack 1 call
- Yield all frames until `checkingdead` is null (the BasicAttack 1 call completed)

## Move 2 - Bubble spray
A party wide bubble spray attack.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoCommand](../../DoCommand.md) calls

|#|Conditions|timer|commandtype|data|
|-:|-----|-----|-----|-----|
|1|None|165.0|[TappingKey](../../Action%20commands/TappingKey.md)|<ul><li>{4.0, 8.0, 0.8<sup>1</sup>, 1.0} if hardmode is false</li><li>{4.0, 8.0, 1.25<sup>1</sup>, 1.0} if hardmode is true</li></ul>

1: Due to the clamping logic related to this value, it means that the decrease value per frame is 0.005 * the game's frametime when hardmode is false and 0.00625 * the game's frametime when hardmode is true. Effectively, it means that hardmode being true makes `barfil` decrease 25% more per frame

### [PartyDamage](../../Damage%20pipeline/PartyDamage.md)

|#|Conditions|caller|damage|property|block|jumpheight|spinammount|jumpevenonblock|overrides|
|-:|---------|-----|-------|-------|-----|----------|-----------|--------------|---------|
|1|Always happen|This enemy|3|null|true only if `barfill` is at least 0.9 after DoCommand 1, false otherwise|0.0|Vector3.zero|false|null|

### Logic sequence

- Camera moves to look near (-2.45, 0.0, 2.45)
- `TidalWormBubbleBeamAudioCue` sound plays
- animstate set to 102
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- animstate set to 103
- `TidalWormBubbleSpray` sound plays
- The `model`'s first child (bubbles particles) gets some adjustmenets on its ParticleSystem:
    - MainModule: startSpeed of 10.0 constant curve and startLifetime of 1.0 constant curve
    - EmissionModule: rateOverTime of 40.0 constant curve
- DoCommand 1 call happens
- All player party member whose `hp` is above 0 has their y `spin` set to 20.0 and their animstate set to 11 (`Hurt`)
- Yield all framed until `doingaction` is false (DoCommand 1 is done)
- All player party member has their `spin` zeroed out
- PartyDamage 1 call happens
- The `model`'s first child (bubbles particles) gets some adjustmenets on its ParticleSystem:
    - MainModule: startSpeed of 5.0 constant curve and startLifetime of 0.75 constant curve
    - EmissionModule: rateOverTime of 20.0 constant curve
- animstate set to 0 (`Idle`)
- Yield for 0.5 seconds

## Move 3 - Enemy summon
Summons a new [Strider](Strider.md) or [DivingSpider](DivingSpider.md) enemy. No damages are dealt.

### Logic sequence

- Camera moves to look near (-1.15, 0.0, 2.0)
- animstate set to 100
- `TidalWormSummon` sound plays
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- All player party member whose `hp` is above 0 has their animstate set to 11 (`Hurt`)
- ShakeScreen called with an ammount of 0.1 and a time of 0.75
- animstate set to 104
- `checkingdead` set to a new [SummonEnemy](../../Actors%20states/Enemy%20party%20members/SummonEnemy.md) call to summon a new [Strider](Strider.md) enemy if `data[0]` is 0 or a [DivingSpider](DivingSpider.md) enemy if it's 1  with type `Offscreen` at (0.5, 0.0, -0.5)
- Yield for 0.5 seconds
- Yield all frames until `checkingdead` is null (the coroutine completed)
- Yield for 0.25 seconds
- `data[0]` is toggled (if it was 0, it becomes 1 and if it was 1, it becomes 0)
- If [flags](../../../Flags%20arrays/flags.md) 166 is true (during a B.O.S.S session in EX mode), the new enemy party member has their `hp` and `maxhp` divided by 2 floored
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) + 1.0 in x
- Yield all frames until `forcemove` is done
- startp is set to this enemy position except the y component where it's 0.0
- `flip` is set to false
