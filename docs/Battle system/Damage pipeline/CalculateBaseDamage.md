# CalculateBaseDamage
This is a method that contains all the damage calculation logic and returns the final amount calculated from the base one given the battle's state. Every damages that goes through the damage pipeline ends up calling this to get the final amount of damage to deal with the only exception being the `NoException` property as it is the only way to bypass this method and deal the amount actually sent. There is no other damage calculation logic in the pipeline with the only exception being the clamping done on [DoDamage](DoDamage.md) after getting the final number. Every other logic involves other matter after the damage has been calculated by this method.

```cs
private int CalculateBaseDamage(MainManager.BattleData? attacker, ref MainManager.BattleData target, int basevalue, bool block, AttackProperty? property, ref bool weaknesshit, ref bool superguarded, DamageOverride[] overrides)
```

## Parameters
All parameters comes directly from [DoDamage](DoDamage.md) (the damageammount becomes the basevalue) with 3 exceptions:

- `block`: This may be overriden to false by DoDamage under certain condition
- `weaknesshit`: This is meant to initially contain false, the method will set this to true if a weakness has been hit
- `superguarded`: This is meant to initially contain false, the method will set this to true if the player sucessfully super blocked
- `property`: If this is `Pierce` or `Flip` while the `target` is a player party member, it is overriden to null at the start of this method which invalidates it

## Damage flow
While the attacker is optional (null means no attacker), the target is mandatory. Additionally, the target cannot be part of the same party as the attacker. This means an enemy party member cannot attack another enemy party member and a player party member cannot attack another player party member. This results in the following damage flows available:

- No attacker to player party member
- No attacker to enemy party member
- Player party member to enemy party member
- Enemy party member to player party member

Each have different implications on the calculations. It is assumed that the target or the attacker is a player party member if it has the `Player` tag. Otherwise, it is assumed to be an enemy party member when not null.

basevalue is the damage amount that will ultimately be returned. This method may modify it for various reasons until it reaches the end where it becomes final.

## Piercing
There is a concept of piercing which has influences on a lot of the damage calculation.

In order for piercing to apply, the following must all be true:

- The target is an enemy party member
- The target doesn't have `AntiPierce` in its `weakness`
- property is `Atleast1pierce` or `Pierce`

If the above isn't fufilled, piercing doesn't apply.

## Calculation pipeline table
The calculation pipeline is very complex and most of it will be illustrated by a table.

Each rows of the table contains an effect to basevalue or other additional effects when applicable. Each row contains 3 columns:

- Attack direction: The damage flow:
    - Enemy to player: Enemy party member to player party member
    - Player to enemy: Player party member to enemy party member
    - Attacker to Any: The attacker exists and the attacker / target can be in either party
    - Any to player: The attacker may or may not exists and the target is a player party member
    - Any to enemy: The attacker may or may not exists and the target is an enemy party member
    - Any to Any: The attacker may or may not exist and the attacker / target can be in either party
- Condition: The conditions required for this effect to process. If a list is presented, all conditions must be true unless stated otherwise
- Damage effect: The effects on basevalue if Condition is true. This may contain other effects than changing basevalue. Anything underlined means it's an effect other than increasing or decreasing meaning it's more significant and its position in the calculation is important

These effects are shown in the exact order they appear. As for the "Main damage" and "Air attack" effects, they are documented in their dedicated sections below as their logic is too complex to document in the main table.

|Attack direction|Condition|Damage effect|
|----------------|---------|-------------|
|Enemy to player|Attacker is a `TANGYBUG` [enemy](../../Enums%20and%20IDs/Enemies.md)|+ 2|
|Attacker to Any|Always processed|- attacker.`tired`|
|Player to enemy|<ul><li>Not in `demomode`</li><li>attacker is `partypointer[0]` (at the front of formation)</li></ul>|+ 1|
|Player to enemy|<ul><li>Not in `demomode`</li><li>attacker has the `Poison` [condition](../Actors%20states/Conditions.md)</li></ul>|+ amount of attacker's `PoisonAttacker` [medals](../../Enums%20and%20IDs/Medal.md)|
|Player to enemy|<ul><li>Not in `demomode`</li><li>attacker.`hp` <= 4</li></ul>|+ amount of attacker's `AttackUp` [medals](../../Enums%20and%20IDs/Medal.md)|
|Enemy to player|<ul><li>Not in `demomode`</li><li>[flags](../../Flags%20arrays/flags.md) 614 and 88 are true (HARDEST is active after chapter 2 ended)</li></ul>|+ 1|
|Enemy to player|Not in `demomode`|+ attacker.`hardatk` TODO: recheck|
|Enemy to player|<ul><li>Not in `demomode`</li><li>`DoublePainReal` [medal](../../Enums%20and%20IDs/Medal.md) is equipped</li></ul>|<u>Clamped from 2 to 99</u>|
|Attacker to any|None|+ attacker.`charge`|
|Any to player|block is true OR `superblockedthisframe` hasn't expired yet|Decreased by:<ul><li>1 if it's a non super block</li></li><li>2 + amount of attacker's `SuperBlock` [medals](../../Enums%20and%20IDs/Medal.md) if it's a super block<sup>1</sup></li></ul>|
|Enemy to player|<ul><li>Target has `Frozen` [condition](../Actors%20states/Conditions.md)</li><li>no `NoIceBreak` override AND (first that applies of the 3 below)</li></ul>|Ice thaw<sup>2</sup>|
|-|<ul><li>target has a `FrostBite` [medal](../../Enums%20and%20IDs/Medal.md)</li><li>`nonphysical` is false</li><li>attacker.`position` isn't `Underground`</li></ul>|A `Freeze` [delayed condition](../Actors%20states/Delayed%20condition.md) is added to the attacker AND a counter of the full damage to the attacker is scheduled (see the final results explanation below for details)</sup>|
|-|<ul><li>target has a `FrostBite` [medal](../../Enums%20and%20IDs/Medal.md)</li><li>The counter effect above didn't apply</li></ul>|<u>Divided by 2 floored</u>|
|-|target has no `FrostBite` [medal](../../Enums%20and%20IDs/Medal.md)|+ 1|
|Any to Any|<ul><li>target has the `Numb` [condition](../Actors%20states/Conditions.md)</li><li>target doesn't have the the `DefenseDown` [condition](../Actors%20states/Conditions.md)</li><li>property isn't `Flip`</li><li>there is no `IgnoreNumb` in the overrides</li><li>piercing doesn't apply</li></ul>|- 1|
|Any to enemy|<ul><li>target.`noexpatstat` is true (cannot happen because this field is UNUSED)</li><li>[flags](../../Flags%20arrays/flags.md) 162 is false (we aren't using the B.O.S.S system and we aren't in a Cave Of Trials session)</li></ul>|- 1 (this effect never happens under normal gameplay)|
|Any to Any|Always occur|Main damage logic and status infliction (see the section below for more details, may contains basevalue changes)|
|Any to enemy|<ul><li>target.battleentity.`height` is above 0.0</li><li>target.`cantfall` is false</li><li>target.`position` is `Flying`</li><li>target.`lockposition` is false</li><li>There are `NoFall` overrides</li></ul> AND (first of the 2 applies)||
|-|<ul><li>target doesn't already have have the `Topple` [conditions](../Actors%20states/Conditions.md)</li><li>target doesn't have the `Sleep`, `Freeze` or `Numb` [conditions](../Actors%20states/Conditions.md)</li><li>target has `ToppleFirst` or `ToppleAirOnly` in its `weakness` </li></ul>|The `Topple` [condition](../Actors%20states/Conditions.md) is added directly to target.`condition` array for 1 turn followed by target.battleentity.`basestate` set to 21 (`Woobly`)|
|-|The above didn't apply|The enemy falls<sup>3</sup>|
|Any to Any|<ul><li>target has the `DefenseUp` [condition](../Actors%20states/Conditions.md)</li><li>Either property:<ul><li>is null OR</li><li>is not `Flip` or `NoExceptions` while piercing doesn't apply</li></ul></li></ul>|- 1|
|Any to Any|<ul><li>target has the `DefenseDown` [condition](../Actors%20states/Conditions.md)</li><li>piercing doesn't apply</li><li>property isn't `NoExceptions`</li><li>At least one of the following is true:<ul><li>[GetDefense](GetDefense.md) returns above 0</li><li>The `DefenseUp` effect above was just processed</li><li>The target is a player party member while HardMode returns true</li></ul></li></ul>|+ 1|
|Attacker to any|<ul><li>Attacker has the `AttackUp` [condition](../Actors%20states/Conditions.md)</li><li>property isn't `NoExceptions`</li></ul>|+ 1|
|Attacker to any|<ul><li>Attacker has the `AttackDown` [condition](../Actors%20states/Conditions.md)</li><li>property isn't `NoExceptions`</li></ul>|- 1|
|Any to player|The `DoublePainReal` [medal](../../Enums%20and%20IDs/Medal.md) is equipped|<u>Multiplied by 1.25 floored</u>, then increased by 1|
|Player to enemy|<ul><li>Attacker has an `AntlionJaws` [medal](../../Enums%20and%20IDs/Medal.md)</li><li>`actionid` is 0 or below TODO: why include 0 ???</li><li>`currentaction` is `Attack`</li><li>Either the target has the `DefenseUp` [condition](../Actors%20states/Conditions.md) OR its [GetDefense](GetDefense.md) is above 0</li><li>Piercing doesn't apply</li></ul>|Decreased by the lowest between the amount of the attacker's `AntlionJaws` [medals](../../Enums%20and%20IDs/Medal.md) and the target's defense which is:<ul><li>[GetDefense](GetDefense.md)'s value if it's above 0</li><li>1 if [GetDefense](GetDefense.md) is 0 AND it has the `DefenseUp` [condition](../Actors%20states/Conditions.md)<sup>4</sup></li><li>0 otherwise</li></ul>|
|Any to enemy|<ul><li>The target has `DefDownOnFlyHard` in its `weakness`</li><li>target.`position` is `Flying`</li><li>HardMode returns true</li></ul>|+ 1|
|Any to Any|target has the `Sturdy` [condition](../Actors%20states/Conditions.md)|- 3|
|Any to player|target has a `Reflection` [condition](../Actors%20states/Conditions.md)|- amount of target's `Reflection` [medals](../../Enums%20and%20IDs/Medal.md)|
|Any to Any|<ul><li>target doesn't have `LimitX10` in its `weakness`</li><li>property is `Atleast1` or `Atleast1pierce`</li></ul>|<u>Clamped from 1 to 99</u>|
|Any to Enemy|<ul><li>target has `LimitX10` in its `weakness`</li><li>property is `Atleast1` or `Atleast1pierce`</li></ul>|- 10|
|Any to Enemy|target has `LimitX10` in its `weakness`|<ul><li>If basevalue is <= 1, <u>basevalue is set to 0</u></li><li>Otheriwise, <u>basevalue is divided by 10 ceiled</u></li></ul>|

1: This always counts as a super block if `superblockedthisframe` hasn't expired yet. A super block also causes the following to happen:
    
- `superblockedthisframe` is reset to 3.0 frames
- superguarded is set to true
- block is overriden to true

For more information on how super blocks are determined, check [GetBlock](../Battle%20flow/Update.md).

2: This is how ice thawing is done (this is done after the sub effect processed):

- [RemoveCondition](../Actors%20states/Conditions%20methods/RemoveCondition.md) is called with the target to remove the `Freeze` [condition](../Actors%20states/Conditions.md)
- [BreakIce](../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#breakice) is called on the target.battleentity
- target.`cantmove` is set to 0 if the target is an enemy party member or to 1 if it's a player party member TODO: this seems strange for an enemy party, recheck

3: This means the attack happened to a toppled flying enemy or the enemy wasn't topplable and was hit while flying meaning it should be dropped to the ground. This is how this happens:

- [RemoveCondition](../Actors%20states/Conditions%20methods/RemoveCondition.md) is called with the target to remove its `Topple` [condition](../Actors%20states/Conditions.md)
- target.battleentity.`bobrange` and `bobspeed` are set to 0.0
- If target.`animid` isn't 208 (this [enemy](../../Enums%20and%20IDs/Enemies.md) id doesn't exist TODO: ???), target.battleentity.`basestate` is set to 0 (`Idle`)
- If target.`eventonfall` is above -1 (it's defined):
    - `calleventnext` is set to target.`eventonfall`
    - target.battleentity.`basestate` is set to 11 (`Hurt`)
- Otherwise (target.`eventonfall` isn't defined):
    - target.battleentity.`droproutine` is set to a new [Drop](../../Entities/EntityControl/EntityControl%20Methods.md#drop) call on target.battleentity

4: Specifically, this means that if the target's [GetDefense](GetDefense.md) is 1 while having the `DefenseUp` [condition](../Actors%20states/Conditions.md), at most 1 `AntLionJaws` will count still (even if 2 are equipped)

## Main damage logic and status infliction
This section always happen and it contains essential logic about damage calculation as well as the processing of the main status effects.

There are 5 potential procedure that this logic can go and it depends on the property. They are all mutually exclusive because it depends on the exact value of property (but the standard one may be done after another).

|Property|Outcome|
|--------|-------|
|null|[DefaultDamageCalc](DefaultDamageCalc.md) is called with the corresponding values for the parameters with pierce being false and def being target.`def`. TODO: study this more thoroughly to know exactly what breaks.|
|`NoExceptions`|Nothing<sup>1</sup>|
|`Flip`|Flip handling that may or may not be followed by Standard damage calculation (see the section below for more details)|
|Any of the following:<ul><li>`Poison`</li><li>`Numb`</li><li>`Numb1Turn`</li><li>`Sleep`</li><li>`Freeze`</li><li>`Fire`</li><li>`InkOnBlock`</li><li>`Ink`</li><li>`Sticky`</li></ul>|Status infliction logic followed by Standard damage calculation (see the sections below for more details)|
|Anything else not mentioned above|Standard damage calculation (see the section below for more details)|

1: There is a technical possibility that a magic weakness hit is processed with this property. For that to happen, the target needs the `Magic` in its `weakness`, the attacker needs to be a player party member and the attack needs to have been caused by `Icefall`, `FrigidCoffin` or `IceRain`. If this case happens, basevalue is incremented and if `commandsuccess` is true (the player sucessfully perfomed the action command), weaknesshit is set to true.

This edge case however should never happen under normal gameplay because these 3 skills do not feature a `NoExceptions` property in their [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md) logic. It is mentioned here because it is otherwise possible in theory, but it is considered invalid. The normal case for a `NoExceptions` is to not do anything in the main damage calculation section.

### Standard damage calculation
This procedure happens if the property is none of the ones mentioned above, but it's also processed in addition to the status infliction property's logic and it may be processed with the `Flip` logic.

It is composed of 2 parts: DefaultDamageCalc and magic weakness processing

#### DefaultDamageCalc 
The first thing that happens is [DefaultDamageCalc](DefaultDamageCalc.md) is called with the corresponding values for the parameters where pierce is true if piercing applies (false otherwise), with def being target.`def`.

#### Magic weakness processing
After, a magic weakness hit test is performed. It succeeds if the target has `Magic` in its `weakness` as well as at least one of the following being true:

- The property is `Magic`
- The attacker is a player party member using the `Icefall`, `FrigidCoffin` or `IceRain` skill

If this test succeeds:

- basevalue is incremented
- If `commandsuccess` is true (the player suceeded the action command), weaknesshit is set to true

### Status infliction
What happens here depends on the property, but they all have one thing in common: after they are processed, the standard damage calculation procedure applies on top of this procedure (see the section above for more details). This is true no matter what happens (even if the infliction fails, succeeds or nothing happens in the first place).

These typically attempts to inflict a [condition](../Actors%20states/Conditions.md) on the target when applicable and most of them implies a resistance check.

Here's an explanation of the columns:

- propert: The property value the row applies to
- [conditions](../Actors%20states/Conditions.md#conditions) inflicted: The condition to inflict if all the requirements are fufilled
- Resistance check?: Whether or not a resistance check must pass for the condition to be inflicted. A resistance is a field on the target that can be at most 99 for the condition to possibly inflict. If it's 100 or above, the target is immune and the infliction won't occur. This immunity condition also affects if the `StatusMirror` [medal](../../Enums%20and%20IDs/Medal.md) works. This is how the test occurs:
    - If this is a `chompyaction` or the attacker is a player party member with a `StatusBoost` [medal](../../Enums%20and%20IDs/Medal.md), the resistance used for the test is decreased by 15 (guaranteed to inflict if this subtraction makes the resistance land at 0 or below)
    - A random number between 0 and 99 is generated and it must be higher or equal than the target's resistance for the test to pass. Intuitively, if the target isn't immune, it means the percentage to inflict is 100 - resistance with `StatusBoost` or a `chompyaction` giving 15% more chances (or a guarantee if the percent number is above 100)
- Requirements: The requirements that must be fufilled for the condition to be inflicted. All statuses requires block to be false and the target to not have the `Sturdy` [condition](../Actors%20states/Conditions.md). Those checks are implied in this column and won't be mentioned. There are 2 requirements specifically that recurs a lot and will be called with a simplified name:
    - Resistance check: A resistance check must pass for the condition to be inflicted. A resistance is an integer field on the target that can be at most 99 for the condition to possibly inflict. If it's 100 or above, the target is immune and the infliction won't occur. This immunity condition also affects if the `StatusMirror` [medal](../../Enums%20and%20IDs/Medal.md) works. This is how the test occurs:
        - If this is a `chompyaction` or the attacker is a player party member with a `StatusBoost` [medal](../../Enums%20and%20IDs/Medal.md), the resistance used for the test is decreased by 15 (guaranteed to inflict if this subtraction makes the resistance land at 0 or below)
        - A random number between 0 and 99 is generated and it must be higher or equal than the target's resistance for the test to pass. Intuitively, if the target isn't immune, it means the percentage to inflict is 100 - resistance with `StatusBoost` or a `chompyaction` giving 15% more chances (or a guarantee if the percent number is above 100)
    - Requires !CanBeToppled?: CanBeToppled needs to return false with the target for the infliction to be allowed. For it to return false, at least one of the following must be true:
        - The target has the `Topple` [conditions](../Actors%20states/Conditions.md)
        - target.`position` is `Flying` while having the `Sleep`, `Freeze` or `Numb` [conditions](../Actors%20states/Conditions.md)
        - The target doesn't have `ToppleFirst` or `ToppleAirOnly` in its `weakness` (if it has the latter, its `position` must not be `Flying`)
- Resistance increase: After infliction, it's possible the resistance used with the resistance check gets increased. This column indicates if it will happen, under what conditions it will happen and for how much
- `StatusMirror` Infliction scheme: The `StatusMirror` [medal](../../Enums%20and%20IDs/Medal.md) scheme of infliction. This medal only works in an enemy to player attack direction and it requires the resistance used for the resistance check to be below 100 (not being immune). If these conditions are met upon inflictions, the attacker gets inflicted with the same condition. The scheme of the infliction may vary and this column tells how it's inflicted
- Other effects: Anything the infliction does other than inflicting the condition or giving it to the attacker due to `StatusMirroe`. Please note that with the exception of the `Sleep` property, [RemoveCondition](../Actors%20states/Conditions%20methods/RemoveCondition.md) is always called with the target to remove its `Sleep` [condition](../Actors%20states/Conditions.md) upon infliction. NOTE: there was supposed to be a condition for removal to happen, but it's broken because the target needs to be an enemy party member OR the player attacker doesn't have the `HeavySleeper` [medal](../../Enums%20and%20IDs/Medal.md), but the latter is impossible if the former is false making this condition always true

|property|[conditions](../Actors%20states/Conditions.md#conditions) inflicted|Requirements|Resistance increase|`StatusMirror` Infliction scheme|Other effects|
|----|------|-----|-----|-----|-----|
|`Poison`|`Poison` for 2 turns|Pass a resistance check with target.`poisonres`|None|[SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with the attacker to set the `Poison` [condition](../Actors%20states/Conditions.md) for 2 turns|The `Poison` sound is played if it wasn't playing already|
|`Numb` or `Numb1Turn`|`Numb` for 2 turns (1 turn instead if it's a `chompyaction` or property is `Numb1Turn`)|<ul><li>Pass a resistance check with target.`numbres`</li><li>Requires !CanBeToppled</li></ul>|If target is an enemy party member, it is increased by 17. The increase is 22 instead if HardMode returns true|AddDelayedCondition is called with the attacker to add the `Numb` [delayed condition](../Actors%20states/Delayed%20condition.md) to it|<ul><li>The `Numb` sound is played if it wasn't playing already</li><li>target.`isnumb` is set to true</li><li>If the target is an enemy party member and its `actimmobile` is false, target.`cantmove` is set to 1 (meaning 1 actor turn needs to pass before the target can act again)</li><li>If target.`position` is `Ground` and its battleentity.`height` is above 0.05, target.battleentity.`droproutine` is set to a new [Drop](../../Entities/EntityControl/EntityControl%20Methods.md#drop) call on target.battleentity</li></ul>|
|`Sleep`|`Sleep` for 2 turns (3 turns instead if the target is a player party member while it already had a `Sleep` condition TODO: this seems backwards...)|<ul><li>Pass a resistance check with target.`sleepres`</li><li>Requires !CanBeToppled</li></ul>|If the target is an enemy party member, target.`numbres` is increased by 9. The increase is 13 instead if HardMode returns true|AddDelayedCondition is called with the attacker to add the `Sleep` [delayed condition](../Actors%20states/Delayed%20condition.md) to it|<ul><li>The `Sleep` sound is played if it wasn't playing already</li><li>target.`isasleep` is set to true</li><li>If the target has the `Flipped` [condition](../Actors%20states/Conditions.md), target.battleentity.[animstate](../../Entities/EntityControl/Animations/animstate.md) is set to 25 (`SleepFallen`) and if it doesn't have it, it's set to 14 (`Sleep`)</li><li>If target.`position` is `Ground` and its battleentity.`height` is above 0.05, target.battleentity.`droproutine` is set to a new [Drop](../../Entities/EntityControl/EntityControl%20Methods.md#drop) call on target.battleentity</li></ul>|
|`Freeze`|`Freeze` for 1 turn if the target is an enemy party member or 2 turns if it's a player party member|<ul><li>Pass a resistance check with target.`freezeres`</li><li>Requires !CanBeToppled</li></ul>|If the target is an enemy party member:<ul><li>If the corresponding [endata](../../TextAsset%20Data/Entity%20data.md#animid-data) of the target's `hasiceanim` is true and we are either at the `GiantLairFridgeInside` [map](../../Enums%20and%20IDs/Maps.md) or in any maps outside of the `GiantLair` [area](../../Enums%20and%20IDs/librarystuff/Areas.md), target.`freezeres` is increased by 70</li><li>Otherwise, if target.`frozenlastturn` is true, target.`freezeres` is increased by 25</li><li>Otherwise, target.`freezeres` is increased by 13. The increase is 18 instead if HardMode returns true</li></ul>|AddDelayedCondition is called with the attacker to add the `Freeze` [delayed condition](../Actors%20states/Delayed%20condition.md) to it|<ul><li>[RemoveCondition](../Actors%20states/Conditions%20methods/RemoveCondition.md) is called with the target to remove the `Topple` [condition](../Actors%20states/Conditions.md)</li><li>If the corresponding [endata](../../TextAsset%20Data/Entity%20data.md#animid-data) of the target's `hasiceanim` is true and we are either at the `GiantLairFridgeInside` [map](../../Enums%20and%20IDs/Maps.md) or in any maps outside of the `GiantLair` [area](../../Enums%20and%20IDs/librarystuff/Areas.md):<ul><li>target.battleentity.`inice` is set to true</li><li>target.`weakness` is set to a new list with one element being `HornExtraDamage`</li></ul></li><li>[Freeze](../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#freeze) is called on target.battleentity</li><li>If target.`position` is `Ground` and its battleentity.`height` is above 0.05, target.battleentity.`droproutine` is set to a new [Drop](../../Entities/EntityControl/EntityControl%20Methods.md#drop) call on target.battleentity</li></ul>|
|`Fire`|`Fire` for 2 turns|CanBeOnFire must returns true<sup>1</sup>|None|[SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with the attacker to set the `Fire` [condition](../Actors%20states/Conditions.md) for 2 turns|The `Flame` sound is played if it wasn't playing already|
|`Ink` or `InkOnBlock` (`InkOnBlock` doesn't have the block requirement)|`Inked` for 3 turns when block is false (2 when block is true)|If the target has the `ResistAll` [medal](../../Enums%20and%20IDs/Medal.md) equipped, a 50% RNG test is performed and it must pass (not applicable if the target doesn't have it equipped)|None|[SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with the attacker to set the `Inked` [condition](../Actors%20states/Conditions.md) for 2 turns|<ul><li>The `WaterSplash2` sound is played at 0.7 pitch if it wasn't playing already or it was while its time is higher than 0.25 seconds</li><li>The `InkGet` particles plays at the target.battleentity's position + Vector3.up</li></ul>|
|`Sticky`|`Sticky` for 3 turns when block is false (2 when block is true)|If the target has the `ResistAll` [medal](../../Enums%20and%20IDs/Medal.md) equipped, a 50% RNG test is performed and it must pass (not applicable if the target doesn't have it equipped)|None|[SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with the attacker to set the `Sticky` [condition](../Actors%20states/Conditions.md) for 2 turns|<ul><li>The `WaterSplash2` sound is played at 0.7 pitch if it wasn't playing already or it was while its time is higher than 0.25 seconds</li><li>The `StickyGet` particles plays at the target.battleentity's position + Vector3.up</li></ul>|

1: The method does the following to determine if the target can be on fire:

- If the target is a player party member, true is returned unless it has the `ResistAll` [medal](../../Enums%20and%20IDs/Medal.md) equipped and a 50% RNG test is failed where false is returned instead
- If it's a `WaspKing` or `EverlastingKing` [enemy](../../Enums%20and%20IDs/Enemies.md), true is returned if a 30% RNG test succeed, false otherwise
- If it's a `Krawler`, `Cape` or `CursedSkull` [enemy](../../Enums%20and%20IDs/Enemies.md) then it's false if the current [map](../../Enums%20and%20IDs/Maps.md) is any in the `GiantLair` [area](../../Enums%20and%20IDs/librarystuff/Areas.md) other than `GiantLairFridgeInside`. Otherwise, the return is decided with a 50% RNG test
- If it's a `KeyR`, `KeyL` or `Tablet` [enemy](../../Enums%20and%20IDs/Enemies.md), false is returned
- If none of the cases applies, true is returned

### Flip handing
This procedure is unique as it is the only property that mar or may not end up using the standard damage calculation procedure.

- If the target doesn't have the `Flipped` [condition](../Actors%20states/Conditions.md), basevalue is decreased by the clamp of target.`def` - 1 from 0 to 99. If the target also has `Flip` in its `weakness`:
    - weaknesshit is set to true
    - If the target doesn't have `ToppleFirst` in its `weakness`:
        - A `Flip` [condition](../Actors%20states/Conditions.md) for 1 turn is added directly to target.`condition`
        - basevalue is clamped from 1 to 99
    - Otherwise, if the target has the `Topple` [condition](../Actors%20states/Conditions.md):
        - A `Flip` [condition](../Actors%20states/Conditions.md) for 1 turn is added directly to target.`condition`
        - basevalue is clamped from 1 to 99
        - [RemoveCondition](../Actors%20states/Conditions%20methods/RemoveCondition.md) is called with the target to remove its `Topple` [condition](../Actors%20states/Conditions.md)
    - Otherwise:
        - If target.`position` is `Ground`, target.battleentity.[animstate](../../Entities/EntityControl/Animations/animstate.md) is set to 21 (`Woobly`)
        - A `Topple` [condition](../Actors%20states/Conditions.md) for 1 turn is added directly to target.`condition`
- If the target has `HornExtraDamage` in its `weakness`:
    - basevalue is incremented
    - weaknesshit is set to true
- If the target is an enemy party member:
    - If target.`holditem` isn't -1 (it was holding an item), [DropItem](../Actors%20states/DropItem.md) is called with the target and with additem
    - If target.`isdefending` is true, it is set to false
- If the target had the `Flipped` [condition](../Actors%20states/Conditions.md) at the start of this procedure, the standard damage calculation procedure is performed (see the section above for details). Otherwise, this procedure ends

## Final steps and results
Before the method ends:

- `caninputcooldown` is set to 0.0
- `blockcooldown` is set to 0.0

The final result is determined by the following:

- If a `FrostBite` [medal](../../Enums%20and%20IDs/Medal.md) counter conditions were fufilled:
    - [ShowDamageCounter](../Visual%20rendering/ShowDamageCounter.md) is called with type 0 (damage) with the basevalue (which is now the final amount) as the ammount starting from the world [CenterPos](../Actors%20states/CenterPos.md) of attacker and ending at Vector3.Up.
    - The attacker's `hp` is decreased by basevalue clamped from 1 to its `maxhp` (meaning this can't be lethal)
    - 0 is returned as the final damage value (this is because the calculations are overriden to deal no damages to the target, but they were dealt manually just now to the attacker)
- Otheriwse, if the attack direction is attacker to player while block is false, basevalue is clamped from 1 to 99
- Otherwise, basevalue is clamped from 0 to 99
