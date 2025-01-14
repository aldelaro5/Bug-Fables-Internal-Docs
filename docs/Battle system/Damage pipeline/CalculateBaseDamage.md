# CalculateBaseDamage
This is a method that contains all the damage calculation logic and returns the final amount calculated from the base one given the battle's state. Every damages that goes through the damage pipeline ends up calling this to get the final amount of damage to deal with the exception of the target being invulnerable which are any of the following conditions on the target:

- Its `plating` is true
- It has the [Shield](../Actors%20states/BattleCondition/Shield.md) condition
- It is a player party member with the [Numb](../Actors%20states/BattleCondition/Numb.md) condition and the `ShockTrooper` [medal](../../Enums%20and%20IDs/Medal.md) equipped

There is no other damage calculation logic in the pipeline with the only exception being the clamping done on [DoDamage](DoDamage.md) after getting the final number. Every other logic involves other matters after the damage has been calculated by this method. Alternatively, actions's logic can perform anything outside of this pipeline.

```cs
private int CalculateBaseDamage(MainManager.BattleData? attacker, ref MainManager.BattleData target, int basevalue, bool block, AttackProperty? property, ref bool weaknesshit, ref bool superguarded, DamageOverride[] overrides)
```

## Parameters
All parameters comes directly from [DoDamage](DoDamage.md) (the damageammount becomes the basevalue) with some exceptions:

- `block`: This may be overriden to false by DoDamage If any of the following are true:
    - The target has the [Freeze](../Actors%20states/BattleCondition/Freeze.md) condition
    - The target has the [Sleep](../Actors%20states/BattleCondition/Sleep.md) condition
    - The target has the [Numb](../Actors%20states/BattleCondition/Numb.md) condition
    - The target is a player party member with the `Berserker` [medal](../../Enums%20and%20IDs/Medal.md) equipped
    - This is a non super block while [flags](../../Flags%20arrays/flags.md) 15 and 615 are true (chapter 1 started after the combat tutorial while FRAMEONE is active) and `overridechallengeblock` is false
- ref `weaknesshit`: This is meant to initially contain false, the method will set this to true if a `Magic` or `Flip` weakness was processed
- ref `superguarded`: This is meant to initially contain false, the method will set this to true if the player sucessfully super blocked
- `property`: The [property](AttackProperty.md) of the attack sent by DoDamage (overriden to null is if it was `None` by DoDamage). If this is `Pierce` or `Flip` while the `target` is a player party member, it is overriden to null at the start of this method which invalidates it

## Damage flow
While the attacker is optional (null means no attacker), the target is mandatory. Additionally, the target cannot be part of the same party as the attacker. This means an enemy party member cannot attack another enemy party member and a player party member cannot attack another player party member. This results in the following damage flows available:

- No attacker to player party member
- No attacker to enemy party member
- Player party member to enemy party member
- Enemy party member to player party member

Each have different implications on the calculations. It is assumed that the target is a player party member if it has the `Player` tag. Otherwise, it is assumed to be an enemy party member. If the attacker exists, it is assumed to be the opposite party of the target.

basevalue is the damage amount that will ultimately be returned. This method may modify it for various reasons until it reaches the end where it becomes final.

## Piercing
There is a concept of piercing which ignores all effects related to enemy party members's defense. Despite the concept of defense having multiple calculation errors, piercing correctly and exhaustively ignores all components of defense.

In order for piercing to apply, the following must all be true:

- The target is an enemy party member (this is specifically disabled for player party members)
- The target doesn't have `AntiPierce` in its [weakness](../Actors%20states/Enemy%20features.md#weakness) (only the `Spuder` [enemy](../../Enums%20and%20IDs/Enemies.md) can get it via [CheckEvent](../Battle%20flow/Update.md) when applicable)
- property is `Atleast1pierce` or `Pierce` (the former is the same as the latter, but the final clamp will be from 1 instead of 0 for any cases not counting a `FrostBite` [medal](../../Enums%20and%20IDs/Medal.md) counter)

If the above isn't fufilled, piercing doesn't apply.

## Calculation pipeline table
The calculation pipeline is very complex and most of it will be illustrated by a table.

Each rows of the table contains an effect to basevalue or other additional effects when applicable. Each row contains 3 columns:

- Attack direction: The damage flow denoted in party to party notation. Here's the party defintions:
    - Any: For the attacker, it may or may not exist from either party. For the target, it can belong to either party
    - Attacker: The attacker must exist, but can be in either party
    - Player: The attacker or target must be a player party member
    - Enemy: The attacker or target must be an enemy player member
- Condition: The conditions required for this effect to process. If a list is presented, all conditions must be true unless stated otherwise
- Damage effect: The effects on basevalue if Condition is true. This may contain other effects than changing basevalue. Anything underlined means it's an effect other than increasing or decreasing meaning it's more significant and its position in the calculation is important

These effects are shown in the exact order they appear.

|Attack direction|Condition|Damage effect|
|----------------|---------|-------------|
|Any to player|block is true OR `superblockedthisframe` hasn't expired yet|Decreased by<sup>11</sup>:<ul><li>1 if it's a non super block</li></li><li>2 + amount of attacker's `SuperBlock` [medals](../../Enums%20and%20IDs/Medal.md) if it's a super block<sup>1</sup></li></ul>|
|Any to Any|property is `Raw`|<u>basevalue is immediately returned with a clamping that depends if blocking happened:</u><ul><li>If blocking happened, the return value is clamped from 0 to 99</li><li>Otherwise, the return value is clamped from 1 to 99</li></ul>|
|Enemy to player|Attacker is a `TANGYBUG` [enemy](../../Enums%20and%20IDs/Enemies.md)|+ 2|
|Attacker to Any|Always processed|- attacker.`tired`|
|Player to enemy|<ul><li>Not in `demomode`</li><li>`currentturn` is `partypointer[0]` (at the front of formation)<sup>9</sup></li></ul>|+ 1|
|Player to enemy|<ul><li>Not in `demomode`</li><li>attacker has the [Poison](../Actors%20states/BattleCondition/Poison.md) condition</li></ul>|+ amount of `playerdata[currentturn]`'s `PoisonAttacker` [medals](../../Enums%20and%20IDs/Medal.md)<sup>9</sup>|
|Player to enemy|<ul><li>Not in `demomode`</li><li>`playerdata[currentturn].hp`<sup>9</sup> <= 4</li></ul>|+ amount of `playerdata[currentturn]`'s `AttackUp` [medals](../../Enums%20and%20IDs/Medal.md)<sup>9</sup>|
|Enemy to player|<ul><li>Not in `demomode`</li><li>[flags](../../Flags%20arrays/flags.md) 614 and 88 are true (HARDEST is active after chapter 2 ended)</li></ul>|+ 1|
|Enemy to player|Not in `demomode`|+ attacker.`hardatk` (the scaled attack from [GetEnemyData](../../TextAsset%20Data/Enemies%20data.md#getenemydata))|
|Enemy to player|<ul><li>Not in `demomode`</li><li>`DoublePainReal` [medal](../../Enums%20and%20IDs/Medal.md) is equipped</li></ul>|<u>Clamped from 2 to 99</u>|
|Attacker to any|Always processed|+ attacker.`charge`|
|Enemy to player|<ul><li>Target has [Freeze](../Actors%20states/BattleCondition/Freeze.md) condition</li><li>No `NoIceBreak` override AND (first that applies of the 3 below)</li></ul>|Ice thaw<sup>2</sup> (after the sub effect is processed, doesn't change basevalue)|
|-|<ul><li>target has a `FrostBite` [medal](../../Enums%20and%20IDs/Medal.md)</li><li>`nonphyscal` is false</li><li>attacker.[position](../Actors%20states/BattlePosition.md) isn't `Underground`</li></ul>|A [Freeze](../Actors%20states/BattleCondition/Freeze.md) condition is added as a [delayed condition](../Actors%20states/Delayed%20condition.md) to the attacker AND a counter of the full damage to the attacker is scheduled (see the final results explanation below for details)</sup>|
|-|<ul><li>target has a `FrostBite` [medal](../../Enums%20and%20IDs/Medal.md)</li><li>The counter effect above didn't apply</li></ul>|<u>Divided by 2 floored</u>|
|-|target has no `FrostBite` [medal](../../Enums%20and%20IDs/Medal.md)|+ 1|
|Any to Any|<ul><li>target has the [Numb](../Actors%20states/BattleCondition/Numb.md) condition</li><li>target doesn't have the the [DefenseDown](../Actors%20states/BattleCondition/DefenseDown.md) condition. NOTE: this is incorrect<sup>5</sup></li><li>property isn't `Flip`. NOTE: This is incorrect<sup>7</sup> and it doesn't account for `NoExceptions` which won't prevent this effect to apply when it should be</li><li>there is no `IgnoreNumb` in the overrides</li><li>piercing doesn't apply</li></ul>|- 1|
|Any to enemy|<ul><li>target.`noexpatstat` is true (cannot happen because this field is UNUSED)</li><li>[flags](../../Flags%20arrays/flags.md) 162 is false (we aren't using the B.O.S.S system and we aren't in a Cave Of Trials session)</li></ul>|- 1 (this effect never happens under normal gameplay)|
|Any to Any|property is one of the following:<ul><li>`Poison`</li><li>`Numb`</li><li>`Numb1Turn`</li><li>`Sleep`</li><li>`Freeze`</li><li>`Fire`</li><li>`InkOnBlock`</li><li>`Ink`</li><li>`Sticky`</li></ul>|Status infliction logic occurs (see the section below for details), no changes to basevalue|
|Any to Any|property is `Flip`|Flip handling logic (see the section below for details), basevalue may change in various ways|
|Any to Any|property is anything except:<ul><li>`NoExceptions` OR</li><li>`Flip` while target doesn't have the [Flipped](../Actors%20states/BattleCondition/Flipped.md) condition. NOTE: If the enemy has [defenseonhit](../Actors%20states/Enemy%20features.md#defenseonhit-and-isdefending) above 0, this is incorrect<sup>7</sup></li></ul>|DefaultDamageCalc (see the section below for details) is called with the corresponding values for the parameters where pierce is true if piercing applies (false otherwise or if property is null), with def being target.`def`. basevalue may decrease due to various defense modifiers|
|Any to Any|<ul><li>target has `Magic` in its [weakness](../Actors%20states/Enemy%20features.md#weakness) AND</li><li>property is `Magic` OR</li> <ul><li>attacker is a player party member while `lastskill` is `Icefall`, `FrigidCoffin` or `IceRain` [skill](../../Enums%20and%20IDs/Skills.md)<sup>10</sup> AND</li><li>property is one of the following:</li><ul><li>`Poison`</li><li>`Numb`</li><li>`Numb1Turn`</li><li>`Sleep`</li><li>`Freeze`</li><li>`Fire`</li><li>`InkOnBlock`</li><li>`Ink`</li><li>`Sticky`</li><li>`Flip` while target has the [Flipped](../Actors%20states/BattleCondition/Flipped.md) condition</li><li>`NoExceptions`</li></ul></ul></ul>|+ 1 and If `commandsuccess` is true (the player suceeded the action command), weaknesshit is set to true|
|Any to enemy|<ul><li>target.battleentity.`height` is above 0.0</li><li>target.[cantfall](../Actors%20states/Enemy%20features.md#cantfall) is false</li><li>target.[position](../Actors%20states/BattlePosition.md) is `Flying`</li><li>target.`lockposition` is false</li><li>There are no `NoFall` overrides</li></ul> AND (first of the 2 applies)||
|-|<ul><li>target doesn't already have have the [Topple](../Actors%20states/BattleCondition/Topple.md) conditions</li><li>target doesn't have the [Sleep](../Actors%20states/BattleCondition/Sleep.md), [Freeze](../Actors%20states/BattleCondition/Freeze.md) or [Numb](../Actors%20states/BattleCondition/Numb.md) conditions</li><li>target has `ToppleFirst` or `ToppleAirOnly` in its [weakness](../Actors%20states/Enemy%20features.md#weakness) </li></ul>|The [Topple](../Actors%20states/BattleCondition/Topple.md) condition is added directly to target.`condition` array for 1 turn followed by target.battleentity.`basestate` set to 21 (`Woobly`)|
|-|The above didn't apply|The enemy falls<sup>3</sup> (no basevalue changes)|
|Any to Any|<ul><li>target has the [DefenseUp](../Actors%20states/BattleCondition/DefenseUp.md) condition</li><li>Either property:<ul><li>is null OR</li><li>is not `Flip` (NOTE: this is incorrect<sup>7</sup>) or `NoExceptions` while piercing doesn't apply</li></ul></li></ul>|- 1|
|Any to Any|<ul><li>target has the [DefenseDown](../Actors%20states/BattleCondition/DefenseDown.md) condition</li><li>piercing doesn't apply</li><li>property isn't `NoExceptions`.</li><li>At least one of the following is true:<ul><li>target.`def` is above 0. NOTE: If the property is `Flip`, this is incorrect if target.`def` is exactly 1<sup>8</sup></li><li>The [DefenseUp](../Actors%20states/BattleCondition/DefenseUp.md) effect above was just processed. NOTE: This is incorrect<sup>5</sup></li><li>The target is a player party member while HardMode returns true. NOTE: This is inconsistent or incorrect<sup>6</sup></li></ul></li></ul>|+ 1|
|Attacker to any|<ul><li>Attacker has the [AttackUp](../Actors%20states/BattleCondition/AttackUp.md) condition</li><li>property isn't `NoExceptions`</li></ul>|+ 1|
|Attacker to any|<ul><li>Attacker has the [AttackDown](../Actors%20states/BattleCondition/AttackDown.md) condition</li><li>property isn't `NoExceptions`</li></ul>|- 1|
|Enemy to player|The `DoublePainReal` [medal](../../Enums%20and%20IDs/Medal.md) is equipped|<u>Multiplied by 1.25 floored</u>, then increased by 1|
|Player to enemy|<ul><li>Attacker has an `AntlionJaws` [medal](../../Enums%20and%20IDs/Medal.md)</li><li>`actionid` is 0 or below (this field is UNUSED so it's always 0)</li><li>[currentchoice](../Player%20UI/Actions.md) is `Attack`</li><li>Either the target has the [DefenseUp](../Actors%20states/BattleCondition/DefenseUp.md) condition OR its target.`def` is above 0</li><li>Piercing doesn't apply</li></ul>|Increased by the lowest between the amount of the attacker's `AntlionJaws` [medals](../../Enums%20and%20IDs/Medal.md) and the target's defense which is:<ul><li>target.`def` if it's above 0</li><li>1 if target.`def` is 0 AND it has the [DefenseUp](../Actors%20states/BattleCondition/DefenseUp.md) condition</li><li>0 otherwise</li></ul>NOTE: This logic is incorrect<sup>4</sup>|
|Any to enemy|<ul><li>The target has `DefDownOnFlyHard` in its [weakness](../Actors%20states/Enemy%20features.md#weakness) (This is specific to a `VenusBoss` [enemy](../../Enums%20and%20IDs/Enemies.md))</li><li>target.[position](../Actors%20states/BattlePosition.md) is `Flying`</li><li>[HardMode](HardMode.md) returns true</li></ul>|+ 1|
|Any to Any|target has the [Sturdy](../Actors%20states/BattleCondition/Sturdy.md) condition|- 3|
|Any to player|target has a [Reflection](../Actors%20states/BattleCondition/Reflection.md) condition|- amount of target's `Reflection` [medals](../../Enums%20and%20IDs/Medal.md)|
|Any to Any|<ul><li>target doesn't have `LimitX10` in its [weakness](../Actors%20states/Enemy%20features.md#weakness)</li><li>property is `Atleast1` or `Atleast1pierce`</li></ul>|<u>Clamped from 1 to 99</u>|
|Any to Enemy|<ul><li>target has `LimitX10` in its [weakness](../Actors%20states/Enemy%20features.md#weakness)</li><li>property is `Atleast1` or `Atleast1pierce`</li></ul>|- 10|
|Any to Enemy|target has `LimitX10` in its [weakness](../Actors%20states/Enemy%20features.md#weakness)|<ul><li>If basevalue is <= 1, <u>basevalue is set to 0</u></li><li>Otheriwise, <u>basevalue is divided by 10 ceiled</u></li></ul>|

1: This always counts as a super block if `superblockedthisframe` hasn't expired yet. A super block also causes the following to happen:
    
- `superblockedthisframe` is reset to 3.0 frames
- superguarded is set to true
- block is overriden to true

For more information on how super blocks are determined, check [GetBlock](../Battle%20flow/GetBlock.md).

2: This is how ice thawing is done (this is done after the sub effect processed):

- [RemoveCondition](../Actors%20states/Conditions%20methods/RemoveCondition.md) is called with the target to remove the [Freeze](../Actors%20states/BattleCondition/Freeze.md) condition
- [BreakIce](../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#breakice) is called on the target.battleentity
- target.`cantmove` is set to 0 if the target is an enemy party member or to 1 if it's a player party member. This allows for the actor to immediately be able to act no matter its party (the enemy party doesn't get their actor turns advanced until the next main turn while the player party have theirs advanced right after enemies). NOTE: This logic is incorrect for enemy party members because if they have a [moves](../Actors%20states/Enemy%20features.md#moves) higher than 1, they will loose all but 1 actor turn they should have available. The correct logic is the same than what [AdvanceTurnEntity](../Battle%20flow/AdvanceTurnEntity.md) does which is setting `cantmove` to -`moves` + 1.

3: This means the attack happened to a toppled flying enemy or the enemy wasn't topplable and was hit while flying meaning it should be dropped to the ground. This is how this happens:

- [RemoveCondition](../Actors%20states/Conditions%20methods/RemoveCondition.md) is called with the target to remove its [Topple](../Actors%20states/BattleCondition/Topple.md) condition
- target.battleentity.`bobrange` and `bobspeed` are set to 0.0
- If target.`animid` (the [enemy](../../Enums%20and%20IDs/Enemies.md) id) isn't 208, target.battleentity.`basestate` is set to 0 (`Idle`). NOTE: This [enemy](../../Enums%20and%20IDs/Enemies.md) id doesn't exist, the logic was supposed to check for target.battleentity.[animid](../../Enums%20and%20IDs/AnimIDs.md) for not being 207 (`CursedSkull`) which would have allowed it to to remain in its charge animation even when it falls
- If target.[eventonfall](../Actors%20states/Enemy%20features.md#eventonfall) is above -1 (it's defined):
    - `calleventnext` is set to target.`eventonfall`
    - target.battleentity.`basestate` is set to 11 (`Hurt`)
- Otherwise (target.[eventonfall](../Actors%20states/Enemy%20features.md#eventonfall) isn't defined):
    - target.battleentity.`droproutine` is set to a new [Drop](../../Entities/EntityControl/EntityControl%20Methods.md#drop) call on target.battleentity

4: The `AntlionJaws` [medal](../../Enums%20and%20IDs/Medal.md) was supposed to ignore a defense for each instance equipped. However, there are 5 cases in which this doesn't work correctly:

- The [DefenseDown](../Actors%20states/BattleCondition/DefenseDown.md) condition is ignored
- The defense gained as a result of the [Numb](../Actors%20states/BattleCondition/Numb.md) condition is ignored
- [defenseonhit](../Actors%20states/Enemy%20features.md#defenseonhit-and-isdefending) is ignored on enemies who supports it
- The [DefenseUp](../Actors%20states/BattleCondition/DefenseUp.md) condition is ignored UNLESS target.`def` is 0 where it works correctly. Specifically, this means that if the target's target.`def` is 1 while having the [DefenseUp](../Actors%20states/BattleCondition/DefenseUp.md) condition, at most 1 `AntLionJaws` will count still (even if 2 are equipped)
- The property is `Flip`. This will cause both the medal and the 1 defense ignore to apply independently for the same reasons, effectively ignoring the defense that was already ignored befrehand

5: This is incorrect to do because the [DefenseDown](../Actors%20states/BattleCondition/DefenseDown.md) effect later can still apply which can make it override the numb defense giving +1 erroneous increase to basevalue. For this issue to reproduce, the `DefenseDown` conditions needs to be fufilled while the numb defense conditions also applies which will effectively count 1 less defense than it should. The defense in the HUD reports the correct number, but the calculations are wrong.

Additionally, if the target has all the [DefenseUp](../Actors%20states/BattleCondition/DefenseUp.md), [DefenseDown](../Actors%20states/BattleCondition/DefenseDown.md) and [Numb](../Actors%20states/BattleCondition/Numb.md) conditions, now numb defense won't apply when the other 2 cancel each other resulting in numb defense not being applied when it should have been. This results in the same error.

6: There are 2 ways to look at this because the intent here is unclear:

- Negative defense on the player is allowed due to making it possible via [medals](../../Enums%20and%20IDs/Medal.md) and [DefenseDown](../Actors%20states/BattleCondition/DefenseDown.md) always apply
- Negative defense shouldn't be allowed in general and medals causes a design issue

The second case is a design bug. The first case isn't consistent because the medals exists no matter if HardMode applies or not, but the [DefenseDown](../Actors%20states/BattleCondition/DefenseDown.md) effect may or may not apply.

Additionally, this doesn't take into consideration the numb defense even if its logic was correct (that it would apply regardless of [DefenseDown](../Actors%20states/BattleCondition/DefenseDown.md)'s presence or not). It now means that the numb defense may or may not be cancelled correctly depending on the difficulty which is a bug.

7: `Flip` always ignore at most 1 of target.`def`, but it also ignores unconditionally other types of defenses on top of this incorrectly:

- [Numb](../Actors%20states/BattleCondition/Numb.md) defense
- [DefenseUp](../Actors%20states/BattleCondition/DefenseUp.md) condition
- [defenseonhit](../Actors%20states/Enemy%20features.md#defenseonhit-and-isdefending) on supported enemies when they are `isdefending`

They are also cumulative: if all 3 are combined, they all get ignored on top of the 1 defense ignored from target.`def` while only at most 1 was supposed to be ignored across all defenses.

8: This is the infamous issue known as the "def bug". If target.`def` is EXACTLY 1 (0 or 2+ are accidentally correct) while it has a [DefenseDown](../Actors%20states/BattleCondition/DefenseDown.md) condition, the only defense left will be ignored, but [DefenseDown](../Actors%20states/BattleCondition/DefenseDown.md) will still apply. Only one of the two should apply because there's only 1 defense available to ignore. [DefenseDown](../Actors%20states/BattleCondition/DefenseDown.md) only requires that target.`def` is above 0.

9: These incorrectly refers to the `currentturn` or the `playerdata[currentturn]` when it should actually be the attacker. This distinction only matters in practive when using the [IceBeemerang](../Player%20actions/Skills/IceBeemerang.md) action because it is the only one where the attacker and the `currenturn` may be different while still using the regular damage pipeline as opposed to [GetMultiDamage](GetMultiDamage.md). That being said, this issue could manifest itself in any action where the user isn't the attacker and the damage pipeline is invoked as usual.

The overall effects are that the damage bonuses (with the exception of `PoisonAttacker`) never checks the actual attacker of this damage call. It means whether or not they apply depends entirely on who used the skill on their actor turn. For example, if the user of `IceBeemerang` is `Bee`, but `Moth` is in front, neither will get the front bonus, but if `Moth` was the user, both members (even `Bee`) would get it. Neither scenarios are correct. The `LastAttack` bonuses works the same way.

However, the `PoisonAttacker` bonus case is affected in a much more complex fashion: the check to apply the bonus is correct, but not its application. The actual attacker still needs to have the [Poison](../Actors%20states/BattleCondition/Poison.md) condition for the bonus to apply, but IF it applies, it's by the amount of `playerdata[currentturn]`'s amount of `PoisonAttacker` equipped, NOT the attacker. It means that it would be possible to create scenarios when using `IceBeemerang` that leads to unexpected bonus applications. Here are the possible combinations:

- `IceBeemerang` is used by someone who has `PoisonAttacker`: The bonus applies to both player party members as long as they have [Poison](../Actors%20states/BattleCondition/Poison.md) even if the other member doesn't have any `PoisonAttacker` so if the other member doesn't have any, but they have `Poison`, the bonus is incorrectly applied (and confusingly, this scenario will even allow the bonus to apply on this attacker even if the user of the move doesn't have `Poison`)
- `IceBeemerang` is used by someone who doesn't have `PoisonAttacker`, but the other player party member does: The bonus never applies for either player party member, even for the one that has `PoisonAttacker` and regardless of whether or not this member has [Poison](../Actors%20states/BattleCondition/Poison.md) so it incorrectly doesn't apply

10: This incorrectly excludes [IceBeemerang](../Player%20actions/Skills/IceBeemerang.md) so this action cannot benefit from the `Magic` weakness bonus

11: Due to the `DoublePainReal` clamp effect later, player blocking is effectively ignored or has its impact greatly reduced because the player blocking effect is located BEFORE the clamp instead of AFTER

## Status infliction
What happens here depends on the property. These typically attempts to inflict a [condition](../Actors%20states/Conditions.md) on the target when applicable and most of them implies a resistance check.

Here's an explanation of the columns:

- property: The property value the row applies to
- [conditions](../Actors%20states/Conditions.md#conditions) inflicted: The condition to inflict if all the requirements are fufilled
- Requirements: The requirements that must be fufilled for the condition to be inflicted. All statuses requires block to be false and the target to not have the [Sturdy](../Actors%20states/BattleCondition/Sturdy.md) condition. Those checks are implied in this column and won't be mentioned. There is 1 requirement specifically that recurs a lot and will be called with a simplified name:
    - Resistance check: A resistance check must pass for the condition to be inflicted. A resistance is an integer field on the target that can be at most 99 for the condition to possibly inflict. If it's 100 or above, the target is immune and the infliction won't occur. This immunity condition also affects if the `StatusMirror` [medal](../../Enums%20and%20IDs/Medal.md) works. This is how the test occurs:
        - If this is a `chompyaction` or the `playerdata[currentturn]` is a player party member with a `StatusBoost` [medal](../../Enums%20and%20IDs/Medal.md), the resistance used for the test is decreased by 15 (guaranteed to inflict if this subtraction makes the resistance land at 0 or below). NOTE: The `StatusBoost` clause checks the amount equipped on whoever is taking their actor turn, NOT the attacker which can be different if the action used is [IceBeemerang](../Player%20actions/Skills/IceBeemerang.md) and the property is `Freeze`
        - A random number between 0 and 99 is generated and it must be higher or equal than the target's resistance for the test to pass. Intuitively, if the target isn't immune, it means the percentage to inflict is 100 - resistance with `StatusBoost` or a `chompyaction` giving 15% more chances (or a guarantee if the percent number is above 100)
- Resistance increase: After infliction, it's possible the resistance used with the resistance check gets increased. This column indicates if it will happen, under what conditions it will happen and for how much
- `StatusMirror` Infliction scheme: The `StatusMirror` [medal](../../Enums%20and%20IDs/Medal.md) scheme of infliction. This medal only works in an enemy to player attack direction and it requires the resistance used for the resistance check to be below 100 (not being immune). If these conditions are met upon inflictions, the attacker gets inflicted with the same condition. The scheme of the infliction may vary and this column tells how it's inflicted
- Other effects: Anything the infliction does other than inflicting the condition or giving it to the attacker due to `StatusMirror`. Please note that with the exception of the `Sleep` property, [RemoveCondition](../Actors%20states/Conditions%20methods/RemoveCondition.md) is called with the target to remove its [Sleep](../Actors%20states/BattleCondition/Sleep.md) condition if one of the following is true:
    - The target is an enemy party member. NOTE: This contradicts the requirements for [DoDamage](DoDamage.md) to do the same because it does not feature a clause for dealing damages above 0. This means that a an attack that does no damage, but still inflicts can remove [Sleep](../Actors%20states/BattleCondition/Sleep.md) without changing the `cantmove` which is different than if DoDamage removed it.
    - The target is a player party member and it doesn't have a `HeavySleeper` [medal](../../Enums%20and%20IDs/Medal.md)

|property|[conditions](../Actors%20states/Conditions.md#conditions) inflicted|Requirements|Resistance increase|`StatusMirror` Infliction scheme|Other effects|
|----|------|-----|-----|-----|-----|
|`Poison`|[Poison](../Actors%20states/BattleCondition/Poison.md) for 2 turns|Pass a resistance check with target.`poisonres`|None|[SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with the attacker to set the [Poison](../Actors%20states/BattleCondition/Poison.md) condition for 2 turns|The `Poison` sound is played if it wasn't playing already|
|`Numb` or `Numb1Turn`|[Numb](../Actors%20states/BattleCondition/Numb.md) for 2 turns (1 turn instead if it's a `chompyaction` or property is `Numb1Turn`)|Pass a resistance check with target.`numbres`|If target is an enemy party member, it is increased by 17. The increase is 22 instead if HardMode returns true|AddDelayedCondition is called with the attacker to add the [Numb](../Actors%20states/BattleCondition/Numb.md) as a [delayed condition](../Actors%20states/Delayed%20condition.md) to it|<ul><li>The `Numb` sound is played if it wasn't playing already</li><li>target.`isnumb` is set to true</li><li>If the target is an enemy party member and its `actimmobile` is false, target.`cantmove` is set to 1 (meaning 1 actor turn needs to pass before the target can act again). NOTE: This means a status cleansing action on an enemy party member will still have them loose their actor turn</li><li>If target.[position](../Actors%20states/BattlePosition.md) is `Ground` and its battleentity.`height` is above 0.05, target.battleentity.`droproutine` is set to a new [Drop](../../Entities/EntityControl/EntityControl%20Methods.md#drop) call on target.battleentity</li></ul>|
|`Sleep`|[Sleep](../Actors%20states/BattleCondition/Sleep.md) for 2 turns (3 turns instead if the target is a player party member while it already had a [Sleep](../Actors%20states/BattleCondition/Sleep.md) condition<sup>2</sup>)|Pass a resistance check with target.`sleepres`|If the target is an enemy party member, target.`numbres` is increased by 9. The increase is 13 instead if HardMode returns true|AddDelayedCondition is called with the attacker to add the [Sleep](../Actors%20states/BattleCondition/Sleep.md) as a [delayed condition](../Actors%20states/Delayed%20condition.md) to it|<ul><li>The `Sleep` sound is played if it wasn't playing already</li><li>target.`isasleep` is set to true</li><li>If the target has the [Flipped](../Actors%20states/BattleCondition/Flipped.md) condition, target.battleentity.[animstate](../../Entities/EntityControl/Animations/animstate.md) is set to 25 (`SleepFallen`) and if it doesn't have it, it's set to 14 (`Sleep`)</li><li>If target.[position](../Actors%20states/BattlePosition.md) is `Ground` and its battleentity.`height` is above 0.05, target.battleentity.`droproutine` is set to a new [Drop](../../Entities/EntityControl/EntityControl%20Methods.md#drop) call on target.battleentity</li></ul>|
|`Freeze`|[Freeze](../Actors%20states/BattleCondition/Freeze.md) for 1 turn if the target is an enemy party member or 2 turns if it's a player party member|Pass a resistance check with target.`freezeres`|If the target is an enemy party member:<ul><li>If the corresponding [endata](../../TextAsset%20Data/Entity%20data.md#animid-data) of the target's `hasiceanim` is true and we are either at the `GiantLairFridgeInside` [map](../../Enums%20and%20IDs/Maps.md) or in any maps outside of the `GiantLair` [area](../../Enums%20and%20IDs/librarystuff/Areas.md), target.`freezeres` is increased by 70</li><li>Otherwise, if target.`frozenlastturn` is true, target.`freezeres` is increased by 25</li><li>Otherwise, target.`freezeres` is increased by 13. The increase is 18 instead if [HardMode](HardMode.md) returns true</li></ul>|AddDelayedCondition is called with the attacker to add the [Freeze](../Actors%20states/BattleCondition/Freeze.md) as a [delayed condition](../Actors%20states/Delayed%20condition.md) to it|<ul><li>[RemoveCondition](../Actors%20states/Conditions%20methods/RemoveCondition.md) is called with the target to remove the [Topple](../Actors%20states/BattleCondition/Topple.md) condition</li><li>If the corresponding [endata](../../TextAsset%20Data/Entity%20data.md#animid-data) of the target's `hasiceanim` is true and we are either at the `GiantLairFridgeInside` [map](../../Enums%20and%20IDs/Maps.md) or in any maps outside of the `GiantLair` [area](../../Enums%20and%20IDs/librarystuff/Areas.md):<ul><li>target.battleentity.`inice` is set to true</li><li>target.[weakness](../Actors%20states/Enemy%20features.md#weakness) is set to a new list with one element being `HornExtraDamage`</li></ul></li><li>[Freeze](../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#freeze) is called on target.battleentity</li><li>If target.[position](../Actors%20states/BattlePosition.md) is `Ground` and its battleentity.`height` is above 0.05, target.battleentity.`droproutine` is set to a new [Drop](../../Entities/EntityControl/EntityControl%20Methods.md#drop) call on target.battleentity</li></ul>|
|`Fire`|[Fire](../Actors%20states/BattleCondition/Fire.md) for 2 turns|CanBeOnFire must returns true<sup>1</sup>|None|[SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with the attacker to set the [Fire](../Actors%20states/BattleCondition/Fire.md) condition for 2 turns|The `Flame` sound is played if it wasn't playing already|
|`Ink` or `InkOnBlock` (`InkOnBlock` doesn't have the block requirement)|[Inked](../Actors%20states/BattleCondition/Inked.md) for 3 turns when block is false (2 when block is true)|If the target has the `ResistAll` [medal](../../Enums%20and%20IDs/Medal.md) equipped, a 50% RNG test is performed and it must pass (not applicable if the target doesn't have it equipped)|None|[SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with the attacker to set the [Inked](../Actors%20states/BattleCondition/Inked.md) condition for 2 turns|<ul><li>The `WaterSplash2` sound is played at 0.7 pitch if it wasn't playing already or it was while its time is higher than 0.25 seconds</li><li>The `InkGet` particles plays at the target.battleentity's position + Vector3.up</li></ul>|
|`Sticky`|[Sticky](../Actors%20states/BattleCondition/Sticky.md) for 3 turns when block is false (2 when block is true)|If the target has the `ResistAll` [medal](../../Enums%20and%20IDs/Medal.md) equipped, a 50% RNG test is performed and it must pass (not applicable if the target doesn't have it equipped)|None|[SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with the attacker to set the [Sticky](../Actors%20states/BattleCondition/Sticky.md) condition for 2 turns|<ul><li>The `WaterSplash2` sound is played at 0.7 pitch if it wasn't playing already or it was while its time is higher than 0.25 seconds</li><li>The `StickyGet` particles plays at the target.battleentity's position + Vector3.up</li></ul>|

1: The method does the following to determine if the target can be on fire:

- If the target is a player party member, true is returned unless it has the `ResistAll` [medal](../../Enums%20and%20IDs/Medal.md) equipped and a 50% RNG test is failed where false is returned instead
- If it's a `WaspKing` or `EverlastingKing` [enemy](../../Enums%20and%20IDs/Enemies.md), true is returned if a 30% RNG test succeed, false otherwise
- If it's a `Krawler`, `Cape` or `CursedSkull` [enemy](../../Enums%20and%20IDs/Enemies.md) then it's false if the current [map](../../Enums%20and%20IDs/Maps.md) is any in the `GiantLair` [area](../../Enums%20and%20IDs/librarystuff/Areas.md) other than `GiantLairFridgeInside`. Otherwise, the return is decided with a 50% RNG test
- If it's a `KeyR`, `KeyL` or `Tablet` [enemy](../../Enums%20and%20IDs/Enemies.md), false is returned
- If none of the cases applies, true is returned

2: This is due to the first of those 3 turns ending immediately making it really + 2 turns in practice which matches all other cases where it's always + 2 turns.

## `Flip` handing
Each rows of the table contains an effect to basevalue or other additional effects when applicable. Each row contains 3 columns:

- target's party: The party of the target (any for either)
- Condition: The conditions required for this effect to process. If a list is presented, all conditions must be true unless stated otherwise
- Effects: The effects on basevalue if Condition is true. This may contain other effects than changing basevalue. Anything underlined means it's an effect other than increasing or decreasing meaning it's more significant and its position in the calculation is important

These effects are shown in the exact order they appear.

|target's party|Condition|Effects|
|--------------|---------|-------------|
|Any|target doesn't have the [Flipped](../Actors%20states/BattleCondition/Flipped.md) condition|basevalue is decreased by the clamp of target.`def` - 1 from 0 to 99|
|Any|<ul><li>target doesn't have the [Flipped](../Actors%20states/BattleCondition/Flipped.md) condition</li><li>target has `Flip` in its [weakness](../Actors%20states/Enemy%20features.md#weakness)</li></ul>|wealnesshit is set to true + the first of 3 applicable effects below (mutually exclusive)|
|-|target doesn't have `ToppleFirst` in its [weakness](../Actors%20states/Enemy%20features.md#weakness)|<ul><li>A [Flipped](../Actors%20states/BattleCondition/Flipped.md) condition for 1 turn is added directly to target.`condition`</li><li><u>basevalue is clamped from 1 to 99</u></li></ul>|
|-|target has the [Topple](../Actors%20states/BattleCondition/Topple.md) condition|<ul><li>A [Flipped](../Actors%20states/BattleCondition/Flipped.md) condition for 1 turn is added directly to target.`condition`</li><li><u>basevalue is clamped from 1 to 99</u></li><li>[RemoveCondition](../Actors%20states/Conditions%20methods/RemoveCondition.md) is called with the target to remove its [Topple](../Actors%20states/BattleCondition/Topple.md) condition</li></ul>|
|-|Neither of the above applied|<ul><li>If target.[position](../Actors%20states/BattlePosition.md) is `Ground`, target.battleentity.[animstate](../../Entities/EntityControl/Animations/animstate.md) is set to 21 (`Woobly`)</li><li>A [Topple](../Actors%20states/BattleCondition/Topple.md) condition for 1 turn is added directly to target.`condition`</li></ul>|
|Any|target has `HornExtraDamage` in its [weakness](../Actors%20states/Enemy%20features.md#weakness)|<ul><li>basevalue is incremented</li><li>weaknesshit is set to true</li></ul>|
|Enemy|Always occur|<ul><li>If target.[holditem](../Actors%20states/Enemy%20features.md#holditem-and-helditem) isn't -1 (it was holding an item), [DropItem](../Actors%20states/Enemy%20party%20members/DropItem.md) is called with the target and with additem</li><li>If target.[isdefending](../Actors%20states/Enemy%20features.md#defenseonhit-and-isdefending) is true, it is set to false</li></ul>|

## DefaultDamageCalc
This is a sub method to CalculateBaseDamage which only gets called for most property values (the exceptions being `NoExceptions` where it's never called and `Flip` where it may or may not be called).

It takes the basevalue as ref and despite its name, it only processes optional decreases of the basevalue due to defensive effects.

```cs
private void DefaultDamageCalc(MainManager.BattleData target, ref int basevalue, bool pierce, bool blocked, int def)
```

### Parameters:

- `target`: The target from CalculateBaseDamage
- ref `basevalue`: The basevalue from CalculateBaseDamage
- pierce: Whether defense pierce applies. This value is computed from CalculateBaseDamage, but overriden to false if `target` is a player party member
- `blocked`: UNUSED (this is block from CalculateBaseDamage, but it is never read in the method)
- `def`: target.`def` from CalculateBaseDamage. (NOTE: this is the base value, not the effective one). Overriden to 0 if `pierce` is true while `target` is an enemy party member

### Effects
Each rows of the table contains an effect to basevalue or other additional effects when applicable. Each row contains 3 columns:

- target's party: The party of the target (any for either)
- Condition: The conditions required for this effect to process. If a list is presented, all conditions must be true unless stated otherwise
- Damage effect: The effects on basevalue if Condition is true. This may contain other effects than changing basevalue. Anything underlined means it's an effect other than increasing or decreasing meaning it's more significant and its position in the calculation is important

These effects are shown in the exact order they appear.

|target's party|Condition|Damage effect|
|--------------|---------|-------------|
|Any|target doesn't have the [Flipped](../Actors%20states/BattleCondition/Flipped.md) condition|- def|
|Player|target has the [Poison](../Actors%20states/BattleCondition/Poison.md) condition|- (amount of target's `PoisonDefender` [medals](../../Enums%20and%20IDs/Medal.md) - amount of target's `ReversePoison` medals)|
|Player|target is `playerdata[battle.partypointer[playerdata.lenght - 1]]` (the furthest back in the player party formation)|- amount of target's `BackSupport` [medals](../../Enums%20and%20IDs/Medal.md)|
|Player|target is `playerdata[battle.partypointer[0]]` (the front in the player party formation)|- amount of target's `FrontSupport` [medals](../../Enums%20and%20IDs/Medal.md)|
|Player|target.`hp` is <= 4|- 2 * amount of target's `DefenseUp` [medals](../../Enums%20and%20IDs/Medal.md)|
|Player|<ul><li>target has the [Sleep](../Actors%20states/BattleCondition/Sleep.md) condition</li><li>target has a `HeavySleeper` [medal](../../Enums%20and%20IDs/Medal.md)</li></ul>|<u>Divided by 2 floored</u>|
|Any|<ul><li>target.[isdefending](../Actors%20states/Enemy%20features.md#defenseonhit-and-isdefending) is true</li><li>pierce is false</li></ul>|- target.[defenseonhit](../Actors%20states/Enemy%20features.md#defenseonhit-and-isdefending)|

## Final steps and results
Before the method ends:

- `caninputcooldown` is set to 0.0
- `blockcooldown` is set to 0.0

The final result is determined by the following:

- If a `FrostBite` [medal](../../Enums%20and%20IDs/Medal.md) counter conditions were fufilled:
    - [ShowDamageCounter](../Visual%20rendering/ShowDamageCounter.md) is called with type 0 (damage) with the basevalue (which is now the final amount) as the ammount starting from the world [CenterPos](../Actors%20states/CenterPos.md) of attacker and ending at Vector3.Up. NOTE: this basevalue may be unclamped here which allows to show a negative number
    - The attacker's `hp` is decreased by basevalue clamped from 1 to its `maxhp` (meaning this can't be lethal). NOTE: basevalue may be unclamped here which allows it to be negative and heal the attacker instead of dealing damage to it (the outer clamp is only clamping the final hp, NOT the number by which it is decreased)
    - 0 is returned as the final damage value (this is because the calculations are overriden to deal no damages to the target, but they were dealt manually just now to the attacker)
- Otheriwse, if the attack direction is attacker to player while block is false, basevalue is clamped from 1 to 99
- Otherwise, basevalue is clamped from 0 to 99
