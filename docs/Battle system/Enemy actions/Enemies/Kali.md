# `Kali`

## Assumptions
It is assumed that there is a [Beetle](Beetle.md) in the enemy party alongside this enemey. For the most part, everything this enemy does is related to `Beetle` so they are meant to be fought together. If multiple `Beetle` exists, the first occurence will be addressed.

It is also assumed that this enemy has an `eventondeath` set to the proper [EventDialogue](../../Battle%20flow/EventDialogues/Kali.md) because this EventDialogue will allow `Beetle` to die with this enemy.

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: This remains at 0 until the first time that the [Beetle](Beetle.md) revive move is used where it is set to 1. When it is 1, the [SetText](../../../SetText/SetText.md) call portion of the move at the start will not be used so it essentially enforces that it only happens once per battle, but the revival will still occur no matter what when the move is used

## Move selection
5 moves are possible:

1. Heals 5 `hp` to [Beetle](Beetle.md)
2. Inflicts the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition to [Beetle](Beetle.md)
3. Inflicts the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition to [Beetle](Beetle.md)
4. Sets [Beetle](Beetle.md)'s `charge` to 2
5. Revives [Beetle](Beetle.md) leaving them with 33.5% floored of their `hp`

Move 5 is always used (and only used) when this is the last `enemydata` left and `reservedata` is not empty.

Move 1 through 4's usage all requires that `Beetle` is present in the enemy party (this should be guaranteed since if they weren't, they would normally be revived by move 5). If he is present, the move usage decision is done based on the following odds:

|Move|Odds|
|1|1/6|
|2|2/6|
|3|2/6|
|4|1/6|

However, move 1 will not be used if `Beetle`'s [HPPercent](../../Actors%20states/HPPercent.md) is 0.9 or higher. If the move is selected and fails this requirement, move 2 is used instead

## Move 1 - Heals [Beetle](Beetle.md)
Heals 5 `hp` to [Beetle](Beetle.md). No damages are dealt

### Logic sequence

- The enemy party member of `Beetle` is obtained
- `Magic` sound plays
- animstate set to 100
- Yield for a second
- [Heal](../../Actors%20states/Heal.md) called to heal `Beetle` for 5 of its `hp`
- Yield for a second
- animstate set to 0 (`Idle`)
- `flip` set to false

## Move 2 - Inflicts [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) to [Beetle](Beetle.md)
Inflicts the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition to [Beetle](Beetle.md) for 2 main turns (effectively 1 main turns since the current main turn advances soon enough). No damages are dealt.

### Logic sequence

- The enemy party member of `Beetle` is obtained
- `Magic` sound plays
- animstate set to 100
- Yield for a second
- `StatUp` sound plays
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition on `Beetle` for 2 main turns (effectively 1 main turns since the current main turn advances soon enough)
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on `Beetle` with type 1 (blue up arrow)
- Yield for a second
- animstate set to 0 (`Idle`)
- `flip` set to false

## Move 3 - Inflicts [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) to [Beetle](Beetle.md)
Inflicts the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition to [Beetle](Beetle.md) for 2 main turns (effectively 1 main turns since the current main turn advances soon enough). No damages are dealt.

### Logic sequence

- The enemy party member of `Beetle` is obtained
- `Magic` sound plays
- animstate set to 101
- Yield for a second
- `StatUp` sound plays
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on `Beetle` for 2 main turns (effectively 1 main turns since the current main turn advances soon enough)
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on `Beetle` with type 0 (red up arrow)
- Yield for a second
- animstate set to 0 (`Idle`)
- `flip` set to false

## Move 4 - Sets [Beetle](Beetle.md)'s `charge` to 2
Sets [Beetle](Beetle.md)'s `charge` to 2. No damages are dealt.

### Logic sequence

- The enemy party member of `Beetle` is obtained
- `Magic` sound plays
- animstate set to 101
- Yield for a second
- `StatUp` sound plays
- `Beetle`'s `charge` is set to 2
- [StatEffect](../../Visual%20rendering/StatEffect.md) called on `Beetle` with type 4 (green up arrow)
- Yield for a second
- animstate set to 0 (`Idle`)
- `flip` set to false

## Move 5 - Revives [Beetle](Beetle.md)
Revives [Beetle](Beetle.md) leaving them with 33.5% floored of their `hp`. No damages are dealt.

### Logic sequence

- If `data[0]` is 0 (this portion of the move wasn't done before in the battle):
    - Yield for 0.5 seconds
    - `data[0]` is set to 1
    - [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
        - text: `commondialogue[159]`
        - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
        - linebreak: `messagebreak`
        - tridimensional: false
        - position: Vector3.zero
        - size: Vector3.one
        - parent: This enemy
        - caller: null
    - Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released
    - Yield for 0.5 seconds
- animstate set to 102
- Yield for 0.66 seconds
- `HealBreath` sound plays
- `HealSmoke` particles plays at this enemy position + (-1.0, 1.5, -0.1) with an alivetime of 5.0
- Yield for 0.5 seconds
- The first `reservedata` is obtained which should always be `Beetle`
- Yield all frames until `Beetle`'s `deathroutine` is null (meaning it completed as it is possible it isn't fully done)
- `Beetle` has ShakeSprite called on them with intensity of 0.1 and frametimer of 30.0
- Yield for 0.5 seconds
- `Beetle` has [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on them
- [ReviveEnemy](../ReviveEnemy.md) called to revive reserveid 0 (should be `Beetle`) with an hppercent of 0.335 without canmove
- Yield for a frame
- [ShowDamageCounter](../../Visual%20rendering/ShowDamageCounter.md) called to show an amount of 1 `hp` starting at `Beetle` position + 1.0 in y and ending at `Beetle` position + 2.0 in y. NOTE: While it shows 1, the real amount healed is percentage based and it is possible that 33.5% floored is higher than 1
- `Heal` sound plays
- Yield all frames until `Beetle` is `onground`
- `Beetle`'s `exp` is set to 0 (avoids gaining double the `exp` by killing them again)
- `Beetle` animstate set to 13 (`BattleIdle`)
- Yield for 0.5 seconds
