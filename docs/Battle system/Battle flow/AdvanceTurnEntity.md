# AdvanceTurnEntity
This is a method only called by [AdvanceMainTurn](../Battle%20flow/Action%20coroutines/AdvanceMainTurn.md) that takes in an actor whose turn will be advanced and a ref to a bool value that will be set to true if there should be a delay after the turn advancement.

```cs
private void AdvanceTurnEntity(ref MainManager.BattleData t, ref bool delay)
```

## Parameters

- `t`: The actor to advance the turn
- `delay`: a dummy ref bool value. It should always have a false value before calling this method as the method only sets it to true in some cases, but never to false

## Procedure
There are 2 parts to this: [Conditions](../Actors%20states/Conditions.md) advances and state advance.

## [Conditions](../Actors%20states/Conditions.md) advances
This part goes through each `condition` and acts on them. This part refers to what happens to each individual condition unless stated otherwise.

There's 2 sections to this: alive advance and last advance.

### Alive advance
This section occur if the actor is alive and it depends on the `BattleCondtion`.

#### [Reflection](../Actors%20states/BattleCondition/Reflection.md)
The turn count of the condition is set to 0.

#### [Poison](../Actors%20states/BattleCondition/Poison.md)
If there is at least one enemy party member:

- If the actor is a player party member with a `ReversePoison` [medal](../../Enums%20and%20IDs/Medal.md) equipped:
    - The actor's `hp` is increased by the actor's `maxhp` * 0.1 rounded to nearest clamped from 1 to 3. NOTE: the nearest rounding has a quirk where if it ends in .5, the even number will be chosen over the odd one no matter if it is actually the lower or higher bound that is correct mathematically
    - If the `Heal` sound isn't playing or its more than halfway done player, it is played
    - [ShowDamageCounter](../Visual%20rendering/ShowDamageCounter.md) is called with type 1 with the damage amount as the amount healed earlier with the start being the battleentity position + (0.0, 1.0, 0.0) and the end being (0.0, 2.0, 0.0)
- Otherwise, [DoDamage](../Damage%20pipeline/DoDamage.md) is called to the actor with no attacker with a `NoExceptions` property without block. The damageammount is the actor's `maxhp` / 10.0 ceiled - 1 and then clamped from 1 to 3 for a player party member and from 1 to 2 for an enemy party member. The call also has these [overrides](../Damage%20pipeline/DamageOverride.md):
    - `NoFall`
    - `NoIceBreak`
    - `FakeAnim`
    - `DontAwake`
    - `IgnoreNumb`
- If the actor's `hp` became 0, battleentity.`overrideanim` is set to false followed by an OverrideOver call being invoked on the battleentity in 1.0 seconds
- The actor's `hp` is clamped from 1 to its `maxhp`
- delay is set to true

#### [Fire](../Actors%20states/BattleCondition/Fire.md)
If there is at least one enemy party member:

- The `Flame` sound is played
- [DoDamage](../Damage%20pipeline/DoDamage.md) is called to the actor with no attacker with a `NoExceptions` property without block. The damageammount is the player party member's `maxhp` / 7.5 ceiled - 1 and then clamped from 2 to 3. The call also has these [overrides](../Damage%20pipeline/DamageOverride.md):
    - `NoFall`
    - `NoIceBreak`
    - `FakeAnim`
    - `DontAwake`
    - `IgnoreNumb`
- If the actor's `hp` became 0, battleentity.`overrideanim` is set to false followed by an OverrideOver call being invoked on the battleentity in 1.0 seconds
- The actor's `hp` is clamped from 1 to its `maxhp`
- delay is set to true

#### [Eaten](../Actors%20states/BattleCondition/Eaten.md)
It is assumed the actor is a player party member here: 

- delay is set to true
- If `eatenby` exists backed by an enemy party member, the player party member's `hp` is above 0 and there is at least one enemy party member:
    - [DoDamage](../Damage%20pipeline/DoDamage.md) is called to the player party member with no attacker with a `NoExceptions` property, a `NoCounter` overrides without block. The damageammount is the player party member's `maxhp` / 10.0 + 1 ceiled and then clamped from 1 to 99
    - [ShowDamageCounter](../Visual%20rendering/ShowDamageCounter.md) is called with type 0 (damage) with the damage amount as the amount with the start being the `eatenby` enemy party member's `cursoroffset` and the end being (0.0, 2.0, 0.0)
    - [Heal](../Actors%20states/Heal.md) is called on the `eatenby` enemy party member for the same damage amount dealt to the player party member
- If the player party member's `hp` is 0 or below while there are at least 1 player party member with an `hp` above 0 while not having an `eatenby`:
    - `eatenkill` is set to true (used for [AdvanceMainTurn](Action%20coroutines/AdvanceMainTurn.md) to handle this later)
    - [EventDialogue](../Battle%20flow/EventDialogue.md) 19 is started and stored in `checkingdead`

#### [Freeze](../Actors%20states/BattleCondition/Freeze.md)

- The `Freeze` sound is played on the battleentity at 1.5 pitch
- battleentity.`shakeice` is set to true
- delay is set to true

#### [Numb](../Actors%20states/BattleCondition/Numb.md)

- delay is set to true
- The `ElecFast` particles are played at the battleentity position + Vector3.up
- A ShakeSprite coroutine is started on the battleentity with the intensity being (0.1, 0.15, 0.0) for 40.0 frames
- The `Shock` sound is played on the battleentity at 1.25 pitch

#### [Sleep](../Actors%20states/BattleCondition/Sleep.md)
Nothing happens if the actor's `hp` is at least its `maxhp`, but if it doesn't:

- A heal amount is determined with a starting value of the actor's `maxhp` * 0.1 ceiled. If the `HeavySleeper` [medal](../../Enums%20and%20IDs/Medal.md) is equipped on the actor (no checks if it's necesarily a player party member), the amount is tripled. On top of this, if it's not a player party member (checked by the battleentity tag not being `Player`), the amount is clamped from 0 to 4
- [ShowDamageCounter](../Visual%20rendering/ShowDamageCounter.md) is called with type 1 (HP) with the amount determined earlier with a start of the actor position + its `cursoroffset` and an end of Vector3.up
- The actor's `hp` is incremented by the amount determined earlier clamped from 0 to the actor's `maxhp`
- The `Heal` sound is played
- delay is set to true

#### [GradualHP](../Actors%20states/BattleCondition/GradualHP.md)
The same than `Sleep`, but the amount to heal and show is always 2.

#### [GradualTP](../Actors%20states/BattleCondition/GradualTP.md)
If instance.`tp` is less than instance.`maxtp`:

- The `Heal2` sound is played
- [ShowDamageCounter](../Visual%20rendering/ShowDamageCounter.md) is called with type 2 (TP) for 2 as the ammount with a start of battleentity position + the actor's `cursoroffset` and an end of Vector3.up
- instance.`tp` is increased by 2 clamped from 0 to instance.`maxtp`
- delay is set to true

From there, the condition turn counter is decremented with some exceptional cases when this doesn't happen:

- The condition is [Eaten](../Actors%20states/BattleCondition/Eaten.md)
- The condition is [Poison](../Actors%20states/BattleCondition/Poison.md) while the `EternalPoison` [medal](../../Enums%20and%20IDs/Medal.md) is equipped
- The condition is [Flipped](../Actors%20states/BattleCondition/Flipped.md) while the turn count on the first `Flipped` condition (this is assumed to be the same one) is 1 or lower

## Last advance
The first thing that happens is `frozenlastturn` is set to whether or not the actor still has the [Freeze](../Actors%20states/BattleCondition/Freeze.md) condition even after the alive update above.

This section only does something if the condition's turn counter just reached 0 and what happens depends on the condition:

- [Numb](../Actors%20states/BattleCondition/Numb.md): `isnumb` is set to false followed by `cantmove` is set to 1 (resets to one actor turn before an action is available, but it will be exhausted later on this actor turn advance)
- [Sleep](../Actors%20states/BattleCondition/Sleep.md): `isasleep` is set to false followed by `cantmove` is set to 1 (resets to one actor turn before an action is available, but it will be exhausted later on this actor turn advance)
- [Freeze](../Actors%20states/BattleCondition/Freeze.md) / [EventStop](../Actors%20states/BattleCondition/EventStop.md): If the battleentity.`icecube` exists, [BreakIce](../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md) is called on the battleentity. Also, `cantmove` is set to 1 (resets to one actor turn before an action is available, but it will be exhausted later on this actor turn advance)
- [Shield](../Actors%20states/BattleCondition/Shield.md): battleentity.`shieldenabled` is set to false
- [AttackUp](../Actors%20states/BattleCondition/AttackUp.md): If the actor's `atkdownloseatkup` is true, an [AttackDown](../Actors%20states/BattleCondition/AttackDown.md) condition is slated to be added later followed by `atkdownonloseatkup` being set to false

## State advance

- All the [conditions](../Actors%20states/Conditions.md) whose turn count reached 0 are removed
- If an [AttackUp](../Actors%20states/BattleCondition/AttackUp.md) condition's turn count reached 0 while `atkdownonloseatkup` was true earlier:
    - [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with [AttackDown](../Actors%20states/BattleCondition/AttackDown.md) for 2 actor turns on the actor
    - The `StatDown` sound is played
    - [StatEffect](../Visual%20rendering/StatEffect.md) is called on the battleentity with type 2 (red arrow down)
- The actor's `isasleep` is set to whether or not the actor has the [Sleep](../Actors%20states/BattleCondition/Sleep.md) condition
- The actor's `isnumb` is set to whether or not the actor has the [Numb](../Actors%20states/BattleCondition/Numb.md) condition
- If `firststrike` is false, the actor's `turnsalive` or `turnssincedeath` is incremented (the one chosen depends on the `hp` being above 0)
- If the actor has the [Freeze](../Actors%20states/BattleCondition/Freeze.md) or [EventStop](../Actors%20states/BattleCondition/EventStop.md) condition or their `isnumb` or `isasleep` is true, their `cantmove` is set to 0 (one action available) if they are a player party member (checked by having the `Player` tag), 1 (one turn before an action is available) if it's an enemy party member
- Otherwise, if the actor's `hp` is above 0 and it's not a `firststrike`:
    - `cantmove` is decremented which passes a turn in the counter
    - If `moreturnnextturn` is above 0, `cantmove` is decreased by it followed by `moreturnnextturn` being set to 0
- Otherwise, if the actor's `hp` is 0, `cantmove` is set to 0
- The actor's `tired` is set to 0
- The actor's `didnothing` is set to false
- The actor's `haspassed` is set to false
