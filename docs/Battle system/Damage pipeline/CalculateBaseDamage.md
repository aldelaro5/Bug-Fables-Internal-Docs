# CalculateBaseDamage
This is a method that contains all the damage calculation logic and returns the final amount calculated from the base one given the battle's state. Every damages that goes through the damage pipeline ends up calling this to get the final amount of damage to deal with the only exception being the `NoException` property as it is the only way to bypass this method and deal the amount actually sent. There is no other damage calculation logic in the pipeline with the only exception being the clamping done on [DoDamage](DoDamage.md) after getting the final number. Every other logic involves other matter after the damage has been calculated by this method.

```cs
private int CalculateBaseDamage(MainManager.BattleData? attacker, ref MainManager.BattleData target, int basevalue, bool block, AttackProperty? property, ref bool weaknesshit, ref bool superguarded, DamageOverride[] overrides)
```

All parameters comes directly from [DoDamage](DoDamage.md) (the damageammount becomes the basevalue) with 3 exceptions:

- `block`: This may be overriden to false by DoDamage under certain condition
- `weaknesshit`: This is meant to initially contain false, the method will set this to true if a weakness has been hit ??? TODO: it's not clear what a weakness is in general
- `superguarded`: This is meant to initially contain false, the method will set this to true if the player sucessfully super blocked

basevalue is the damage amount that will ultimately be returned. This method may modify it for various reasons until it reaches the end where it becomes final.

The following is the procedure of the damage calculation divided in ordered section TODO: very WIP.

It is assumed that the target or the attacker is a player party member if it has the `Player` tag. Otherwise, it is assumed to be an enemy party member when not null.

## Invalidation the `Pierce` and `Flip` properties
If the target is a player party member and the property is either `Pierce` or `Flip`, it is overriden to null which invalidates it.

## `TANGYBUG` [enemy](../../Enums%20and%20IDs/Enemies.md) attacker specific
If the target is a player party member and the attacker is a `TANGYBUG` [enemy](../../Enums%20and%20IDs/Enemies.md), basevalue is increased by 2.

## Preliminary attacker's attack processing
This section happens only if there is an attacker and it applies several bonuses depending on the attacker's attack.

The following are applied in order when they apply.

### Exhaustion
basevalue is always decremented by the attacker's `tired`.

### Player attacker
This part only applies if the attacker is a player party member and we aren't in `demomode`.

#### Front position bonus
If `partypointer[0]` is `currentturn` (meaning the front player party member is effectively the attacker as it's also the one spending its actor turn), basevalue is incremented.

#### `PoisonAttacker` [medal](../../Enums%20and%20IDs/Medal.md) processing
If the attacker has the `Poison` [condition](../Actors%20states/Conditions.md), basevalue is increased by the amount of `PoisonAttacker` [medals](../../Enums%20and%20IDs/Medal.md) equipped on `playerdata[currentturn]` (the attacker).

#### `AttackUp` [medal](../../Enums%20and%20IDs/Medal.md) processing
If the `playerdata[currentturn]` (the attacker)'s `hp` is 4 or less, basevalue is increased by the amount of `AttackUp` [medals](../../Enums%20and%20IDs/Medal.md) equipped on `playerdata[currentturn]` (the attacker).

### Enemy attacker
This part only applies if the attacker is an enemy party member and we aren't in `demomode`.

#### HARDEST attack bonus
If [flags](../../Flags%20arrays/flags.md) 614 and 88 are true (HARDEST is active after chapter 2 ended), basevalue is incremented.

#### `hardatk`
basevalue is always increased by the attacker's `hardatk`. TODO: odd, recheck this later

#### `DoublePainReal` [medal](../../Enums%20and%20IDs/Medal.md)'s clamp
If the `DoublePainReal` [medal](../../Enums%20and%20IDs/Medal.md) is equipped, basevalue is clamped from 2 to 99.

### `charge` bonus
basevalue is always incremented by the attacker's `charge`.

## Block processing
This section applies only if the target is a player party member while either block is true or `superblockedthisframe` hasn't expired yet (see [GetBlock](../Battle%20flow/Update.md#getblock) for more information on how this works).

### Non super block defense
For a non super block (and `superblockedthisframe` expired), the only logic is basevalue gets decremented.

### Super block defense
For a super block (or `superblockedthisframe` hasn't expired yet):

- `superblockedthisframe` is reset to 3.0 frames
- superguarded is set to true
- block is overriden to true
- basevalue is decreased by 2

#### `SuperBlock` [medal](../../Enums%20and%20IDs/Medal.md) processing
On top of the regular super block decrease, basevalue gets decreased by the amount of `SuperBlock` [medals](../../Enums%20and%20IDs/Medal.md) equipped on the target.

## Frozen target processing
This section happens if the target has the `Freeze` [condition](../Actors%20states/Conditions.md) unless a `NoIceBreak` overrides exists.

### `FrostBite` [medal](../../Enums%20and%20IDs/Medal.md) processing
If the target is a player party member with a `FrostBite` [medal](../../Enums%20and%20IDs/Medal.md) equipped and there is an attacker (assumed to be an enemy party member), then the medal overrides the logic regarding basevalue in 2 ways (they are mutually exclusive).

If this doesn't apply, the standard logic does (see the section below).

#### Physical attack from an above ground attacker
If `nonphysical` is false and the attacker's `position` isn't `Underground`, a `Freeze` [delayed condition](../Actors%20states/Delayed%20condition.md) is added to the attacker with no changes to the basevalue for now. 

However, towards the end of the method, there will be a full counter of the basevalue back to the attacker. See the later section about this for details.

#### `nonphysical` attack or the attacker is `Underground`
This applies if the logic above didn't. In this case, basevalue gets halved via an integer division which is effectively a divide by 2 floored.

### Standard freeze bonus
This logic applies if the `Frostbite` [medal](../../Enums%20and%20IDs/Medal.md) logic didn't.

The only thing that happens is basevalue gets incremented.

### Ice thaw processing
This logic always happen no matter how basevalue was managed just now.

- [RemoveCondition](../Actors%20states/Conditions%20methods/RemoveCondition.md) is called with the target to remove the `Freeze` [condition](../Actors%20states/Conditions.md)
- [BreakIce](../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#breakice) is called on the target.battleentity
- target.`cantmove` is set to 0 if the target is an enemy party member or to 1 if it's a player party member TODO: this seems strange for an enemy party, recheck

## `Numb` [condition](../Actors%20states/Conditions.md) defense
This section only happens if all of the following is true:

- The target has the `Numb` [condition](../Actors%20states/Conditions.md)
- The target doesn't have the the `DefenseDown` [condition](../Actors%20states/Conditions.md)
- The property isn't `Flip`
- There is no `IgnoreNumb` in the overrides
- At least one of the following is false (piercing doesn't apply):
    - The target is an enemy party member
    - The target doesn't have `AntiPierce` in its `weakness`
    - The property is `Atleast1pierce` or `Pierce`

If all of the above is fufilled, basevalue is decremented.

## Dead `noexpatstart` logic
There is logic here that never happens because it requires the target's `noexpatstart` to be true, but this can't happen because this field is never written to. The other 2 requirements were that the target is an enemy party member and that [flags](../../Flags%20arrays/flags.md) 162 is false (we aren't using the B.O.S.S system and we aren't in a Cave Of Trials session).

The logic would have been to decrement the basevalue. TODO: mysterious...

## Main damage and status infliction
This section always happen and it contains essential logic about damage calculation as well as the processing of the main status effects.

There are 5 potential procedure that this logic can go and it depends on the property. They are all mutually exclusive because it depends on the exact value of property (but the standard one may be done after another).

### Property is null
This procedure is simplified to calling [DefaultDamageCalc](DefaultDamageCalc.md) with the corresponding values for the parameters with pierce being false and def being target.`def`.

NOTE: target.`def` does not report the effective defense here. It only reports the fixed base one. This implies many calculation bugs. TODO: study this more thoroughly to know exactly what breaks.

### Property is `NoExceptions`
This procedure doe nothing.

However, there is a technical possibility that a magic weakness hit is processed with this property. For that to happen, the target needs the `Magic` in its `weakness`, the attacker needs to be a player party member and the attack needs to have been caused by `Icefall`, `FrigidCoffin` or `IceRain`. If this case happens, basevalue is incremented and if `commandsuccess` is true (the player sucessfully perfomed the action command), weaknesshit is set to true.

This edge case however should never happen under normal gameplay because these 3 skills do not feature a `NoExceptions` property in their [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md) logic. It is mentioned here because it is otherwise possible in theory, but it is considered invalid. The normal case for a `NoExceptions` is to not do anything in the main damage calculation section.

### Property inflicts one of the main status effects
This procedure applies to any of the following property values:

- `Poison`
- `Numb`
- `Numb1Turn`
- `Sleep`
- `Freeze`
- `Fire`
- `InkOnBlock`
- `Ink`
- `Sticky`

What happens here depends on the property, but they all have one thing in common: after they are processed, the standard damage calculation procedure applies on top of this procedure (see the section below for more details on the standard damage calculation procedure). This is true no matter what happens (even if the infliction fails, succeeds or nothing happens in the first place).

These typically attempts to inflict a [condition](../Actors%20states/Conditions.md) on the target when applicable and most of them implies a resistance check.

#### `Poison`
For this to inflict, the following must be true:

- The target doesn't have the `Sturdy` [condition](../Actors%20states/Conditions.md)
- block is false
- target.`poisonres` is less than 100 (it's not immune)
- A resistance test must pass

For the resistance test, it involves generating a number between 0 and 99 and it must be higher or equal than the poison resistance. This resistance is normally target.`poisonres`, but if this is a `chompyaction` or the attacker is a player party member with a `StatusBoost` [medal](../../Enums%20and%20IDs/Medal.md), it's target.`poisonres` - 15 (guaranteed to inflict if this subtraction makes the resistance land at 0 or below). Intuitively, if the target isn't immune, it means the percentage to inflict is 100 - target.`poisonres` with `StatusBoost` or a `chompyaction` giving 15% more chances (or a guarantee if the percent number is above 100).

If the above conditions are fufilled:

- The `Poison` sound is played if it wasn't playing already
- If the target is an enemy party member or the attacker is a player party member without a `HeavySleeper` [medal](../../Enums%20and%20IDs/Medal.md) equipped, [RemoveCondition](../Actors%20states/Conditions%20methods/RemoveCondition.md) is called with the target to remove the `Sleep` [condition](../Actors%20states/Conditions.md) TODO: I don't know what this means...
- [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with the target to set the `Poison` [condition](../Actors%20states/Conditions.md) for 2 turns
- If there is an attacker with a `poisonres` below 100 (not immune) and the target is a player party member with a `StatusMirror` [medal](../../Enums%20and%20IDs/Medal.md) equipped, [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with the attacker to set the `Poison` [condition](../Actors%20states/Conditions.md) for 2 turns

#### `Numb`
For this to inflict, the following must be true:

- The target doesn't have the `Sturdy` [condition](../Actors%20states/Conditions.md)
- block is false
- target.`numbres` is less than 100 (it's not immune)
- The target cannot be toppled meaning at least one of the following must be true:
    - The target has the `Topple` [conditions](../Actors%20states/Conditions.md)
    - target.`position` is `Flying` while having the `Sleep`, `Freeze` or `Numb` [conditions](../Actors%20states/Conditions.md)
    - The target doesn't have `ToppleFirst` or `ToppleAirOnly` in its `weakness` (if it has the latter, its `position` must not be `Flying`)
- A resistance test must pass

For the resistance test, it involves generating a number between 0 and 99 and it must be higher or equal than the poison resistance. This resistance is normally target.`numbres`, but if this is a `chompyaction` or the attacker is a player party member with a `StatusBoost` [medal](../../Enums%20and%20IDs/Medal.md), it's target.`numbres` - 15 (guaranteed to inflict if this subtraction makes the resistance land at 0 or below). Intuitively, if the target isn't immune, it means the percentage to inflict is 100 - target.`numbres` with `StatusBoost` or a `chompyaction` giving 15% more chances (or a guarantee if the percent number is above 100).

If the above conditions are fufilled:

- The `Numb` sound is played if it wasn't playing already
- If the target is an enemy party member or the attacker is a player party member without a `HeavySleeper` [medal](../../Enums%20and%20IDs/Medal.md) equipped, [RemoveCondition](../Actors%20states/Conditions%20methods/RemoveCondition.md) is called with the target to remove the `Sleep` [condition](../Actors%20states/Conditions.md) TODO: I don't know what this means...
- [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with the target to set the `Numb` [condition](../Actors%20states/Conditions.md) for 2 turns (1 turn instead if it's a `chompyaction`)
- target.`isnumb` is set to true
- If the target is an enemy party member and its `actimmobile` is false, target.`cantmove` is set to 1 (meaning 1 actor turn needs to pass before the target can act again)
- If target.`position` is `Ground` and its battleentity.`height` is above 0.05, target.battleentity.`droproutine` is set to a new [Drop](../../Entities/EntityControl/EntityControl%20Methods.md#drop) call on target.battleentity
- If the target is a player party member:
    - If there is an attacker with a `poisonres` below 100 (not immune) and the target has a `StatusMirror` [medal](../../Enums%20and%20IDs/Medal.md) equipped, AddDelayedCondition is called with the attacker to add the `Numb` [delayed condition](../Actors%20states/Delayed%20condition.md) to it
- Otherwise (the target is an enemy party member):
    - target.`numbres` is increased by 17. The increase is 22 instead if HardMode returns true meaning any of the following is true:
        - The `DoublePain` [medal](../../Enums%20and%20IDs/Medal.md) is equipped
        - [flags](../../Flags%20arrays/flags.md) 614 is true (HARDEST is active)
        - [flags](../../Flags%20arrays/flags.md) 166 is true (EX mode is active on the B.O.S.S system)

#### `Numb1Turn`
The same as `Numb`, but the [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) call always inflict for 1 turn regardless if it's a `chompyaction` or not.

#### `Sleep`
For this to inflict, the following must be true:

- The target doesn't have the `Sturdy` [condition](../Actors%20states/Conditions.md)
- block is false
- target.`sleepres` is less than 100 (it's not immune)
- The target cannot be toppled meaning at least one of the following must be true:
    - The target has the `Topple` [conditions](../Actors%20states/Conditions.md)
    - target.`position` is `Flying` while having the `Sleep`, `Freeze` or `Numb` [conditions](../Actors%20states/Conditions.md)
    - The target doesn't have `ToppleFirst` or `ToppleAirOnly` in its `weakness` (if it has the latter, its `position` must not be `Flying`)
- A resistance test must pass

For the resistance test, it involves generating a number between 0 and 99 and it must be higher or equal than the poison resistance. This resistance is normally target.`sleepres`, but if this is a `chompyaction` or the attacker is a player party member with a `StatusBoost` [medal](../../Enums%20and%20IDs/Medal.md), it's target.`sleepres` - 15 (guaranteed to inflict if this subtraction makes the resistance land at 0 or below). Intuitively, if the target isn't immune, it means the percentage to inflict is 100 - target.`sleepres` with `StatusBoost` or a `chompyaction` giving 15% more chances (or a guarantee if the percent number is above 100).

If the above conditions are fufilled:

- The `Sleep` sound is played if it wasn't playing already
- [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with the target to set the `Sleep` [condition](../Actors%20states/Conditions.md) for 2 turns. It will be for 3 turns instead if the target is a player party member while it already had a `Sleep` condition TODO: this seems backwards...
- target.`isasleep` is set to true
- If the target has the `Flipped` [condition](../Actors%20states/Conditions.md), target.battleentity.[animstate](../../Entities/EntityControl/Animations/animstate.md) is set to 25 (`SleepFallen`) and if it doesn't have it, it's set to 14 (`Sleep`)
- If target.`position` is `Ground` and its battleentity.`height` is above 0.05, target.battleentity.`droproutine` is set to a new [Drop](../../Entities/EntityControl/EntityControl%20Methods.md#drop) call on target.battleentity
- If the target is a player party member:
    - If there is an attacker with a `sleepres` below 100 (not immune) and the target has a `StatusMirror` [medal](../../Enums%20and%20IDs/Medal.md) equipped, AddDelayedCondition is called with the attacker to add the `Sleep` [delayed condition](../Actors%20states/Delayed%20condition.md) to it
- Otherwise (the target is an enemy party member):
    - target.`numbres` is increased by 9. The increase is 13 instead if HardMode returns true meaning any of the following is true:
        - The `DoublePain` [medal](../../Enums%20and%20IDs/Medal.md) is equipped
        - [flags](../../Flags%20arrays/flags.md) 614 is true (HARDEST is active)
        - [flags](../../Flags%20arrays/flags.md) 166 is true (EX mode is active on the B.O.S.S system)

#### `Freeze`
For this to inflict, the following must be true:

- The target doesn't have the `Sturdy` [condition](../Actors%20states/Conditions.md)
- block is false
- target.`freezeres` is less than 100 (it's not immune)
- The target cannot be toppled meaning at least one of the following must be true:
    - The target has the `Topple` [conditions](../Actors%20states/Conditions.md)
    - target.`position` is `Flying` while having the `Sleep`, `Freeze` or `Numb` [conditions](../Actors%20states/Conditions.md)
    - The target doesn't have `ToppleFirst` or `ToppleAirOnly` in its `weakness` (if it has the latter, its `position` must not be `Flying`)
- A resistance test must pass

For the resistance test, it involves generating a number between 0 and 99 and it must be higher or equal than the poison resistance. This resistance is normally target.`freezeres`, but if this is a `chompyaction` or the attacker is a player party member with a `StatusBoost` [medal](../../Enums%20and%20IDs/Medal.md), it's target.`freezeres` - 15 (guaranteed to inflict if this subtraction makes the resistance land at 0 or below). Intuitively, if the target isn't immune, it means the percentage to inflict is 100 - target.`freezeres` with `StatusBoost` or a `chompyaction` giving 15% more chances (or a guarantee if the percent number is above 100).

If the above conditions are fufilled:

- If the target is an enemy party member or the attacker is a player party member without a `HeavySleeper` [medal](../../Enums%20and%20IDs/Medal.md) equipped, [RemoveCondition](../Actors%20states/Conditions%20methods/RemoveCondition.md) is called with the target to remove the `Sleep` [condition](../Actors%20states/Conditions.md) TODO: I don't know what this means...
- [RemoveCondition](../Actors%20states/Conditions%20methods/RemoveCondition.md) is called with the target to remove the `Topple` [condition](../Actors%20states/Conditions.md)
- [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with the target to set the `Freeze` [condition](../Actors%20states/Conditions.md) for 1 turn if the target is an enemy party member and 2 turns if it's a player party member.
- If the target is a player party member:
    - If there is an attacker with a `freezeres` below 100 (not immune) and the target has a `StatusMirror` [medal](../../Enums%20and%20IDs/Medal.md) equipped, AddDelayedCondition is called with the attacker to add the `Freeze` [delayed condition](../Actors%20states/Delayed%20condition.md) to it
- Otherwise (the target is an enemy party member), what happens depends on some conditions (mutually exclusive, only the first one applies):
    - If the corresponding [endata](../../TextAsset%20Data/Entity%20data.md#animid-data) of the target's `hasiceanim` is true and we are either at the `GiantLairFridgeInside` [map](../../Enums%20and%20IDs/Maps.md) or in any maps outside of the `GiantLair` [area](../../Enums%20and%20IDs/librarystuff/Areas.md):
        - target.`freezeres` is increased by 70
        - target.battleentity.`inice` is set to true
        - target.`weakness` is set to a new list with one element being `HornExtraDamage`
    - Otherwise, if target.`frozenlastturn` is true, target.`freezeres` is increased by 25
    - If none of the above applied, target.`freezeres` is increased by 13. The increase is 18 instead if HardMode returns true meaning any of the following is true:
        - The `DoublePain` [medal](../../Enums%20and%20IDs/Medal.md) is equipped
        - [flags](../../Flags%20arrays/flags.md) 614 is true (HARDEST is active)
        - [flags](../../Flags%20arrays/flags.md) 166 is true (EX mode is active on the B.O.S.S system)
- [Freeze](../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#freeze) is called on target.battleentity
- If target.`position` is `Ground` and its battleentity.`height` is above 0.05, target.battleentity.`droproutine` is set to a new [Drop](../../Entities/EntityControl/EntityControl%20Methods.md#drop) call on target.battleentity

#### `Fire`
For this to inflict, the following must be true:

- The target doesn't have the `Sturdy` [condition](../Actors%20states/Conditions.md)
- block is false
- The target CanBeOnFire test pass

CanBeOnFire is a method that returns true if the target allows the infliction. Here is what it does:

- If the target is a player party member, true is returned unless it has the `ResistAll` [medal](../../Enums%20and%20IDs/Medal.md) equipped and a 50% RNG test is failed where false is returned instead
- If it's a `WaspKing` or `EverlastingKing` [enemy](../../Enums%20and%20IDs/Enemies.md), true is returned if a 30% RNG test succeed, false otherwise
- If it's a `Krawler`, `Cape` or `CursedSkull` [enemy](../../Enums%20and%20IDs/Enemies.md) then it's false if the current [map](../../Enums%20and%20IDs/Maps.md) is any in the `GiantLair` [area](../../Enums%20and%20IDs/librarystuff/Areas.md) other than `GiantLairFridgeInside`. Otherwise, the return is decided with a 50% RNG test
- If it's a `KeyR`, `KeyL` or `Tablet` [enemy](../../Enums%20and%20IDs/Enemies.md), false is returned
- If none of the cases applies, true is returned

If the above conditions are fufilled:

- The `Flame` sound is played if it wasn't playing already
- If the target is an enemy party member or the attacker is a player party member without a `HeavySleeper` [medal](../../Enums%20and%20IDs/Medal.md) equipped, [RemoveCondition](../Actors%20states/Conditions%20methods/RemoveCondition.md) is called with the target to remove the `Sleep` [condition](../Actors%20states/Conditions.md). TODO: I don't know what this means...
- [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with the target to set the `Fire` [condition](../Actors%20states/Conditions.md) for 2 turns
- If there is an attacker and the target is a player party member with a `StatusMirror` [medal](../../Enums%20and%20IDs/Medal.md) equipped, [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with the attacker to set the `Fire` [condition](../Actors%20states/Conditions.md) for 2 turns

#### `InkOnBlock`
For this to inflict, the following must be true:

- The target doesn't have the `Sturdy` [condition](../Actors%20states/Conditions.md)
- If the target has the `ResistAll` [medal](../../Enums%20and%20IDs/Medal.md) equipped, a 50% RNG test is performed and it must pass (not applicable if the target doesn't have it equipped)

If the above conditions are fufilled:

- The `WaterSplash2` sound is played at 0.7 pitch if it wasn't playing already or it was while its time is higher than 0.25 seconds
- The `InkGet` particles plays at the target.battleentity's position + Vector3.up
- If the target is an enemy party member or the attacker is a player party member without a `HeavySleeper` [medal](../../Enums%20and%20IDs/Medal.md) equipped, [RemoveCondition](../Actors%20states/Conditions%20methods/RemoveCondition.md) is called with the target to remove the `Sleep` [condition](../Actors%20states/Conditions.md). NOTE: this cannot happen under normal gameplay because there are no way to directly inflict `Inked` from a player's action TODO: I don't know what this means...
- [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with the target to set the `Inked` [condition](../Actors%20states/Conditions.md) for 3 turns when block is false (2 when block is true)
- If there is an attacker and the target is a player party member with a `StatusMirror` [medal](../../Enums%20and%20IDs/Medal.md) equipped, [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with the attacker to set the `Inked` [condition](../Actors%20states/Conditions.md) for 2 turns

#### `Ink`
If block is true, nothing happens (only the standard damage calculation procedure applies as mentioned above).

Otherwise, it's the same procedure than `InkOnBlock`.

#### `Sticky`
For this to inflict, the following must be true:

- The target doesn't have the `Sturdy` [condition](../Actors%20states/Conditions.md)
- block is false
- If the target has the `ResistAll` [medal](../../Enums%20and%20IDs/Medal.md) equipped, a 50% RNG test is performed and it must pass (not applicable if the target doesn't have it equipped)

If the above conditions are fufilled:

- The `WaterSplash2` sound is played at 0.8 pitch if it wasn't playing already or it was while its time is higher than 0.25 seconds
- The `StickyGet` particles plays at the target.battleentity's position + Vector3.up
- If the target is an enemy party member or the attacker is a player party member without a `HeavySleeper` [medal](../../Enums%20and%20IDs/Medal.md) equipped, [RemoveCondition](../Actors%20states/Conditions%20methods/RemoveCondition.md) is called with the target to remove the `Sleep` [condition](../Actors%20states/Conditions.md). NOTE: this cannot happen under normal gameplay because there are no way to directly inflict `Sticky` from a player's action TODO: I don't know what this means...
- [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with the target to set the `Sticky` [condition](../Actors%20states/Conditions.md) for 3 turns when block is false (2 when block is true)
- If there is an attacker and the target is a player party member with a `StatusMirror` [medal](../../Enums%20and%20IDs/Medal.md) equipped, [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with the attacker to set the `Sticky` [condition](../Actors%20states/Conditions.md) for 2 turns

### Property is `Flip`
This procedure is unique as it is the only property that mar or may not end up using the standard damage calculation procedure.

- If the target doesn't have the `Flipped` [condition](../Actors%20states/Conditions.md), basevalue is decreased by the clamp of target.`def` - 1 from 0 to 99. If the target also has the `Flip` in its `weakness`:
    - weaknesshit is set to true
    - If the target doesn't have `ToppleFirst` in its `weakness`:
        - A `Flip` [condition](../Actors%20states/Conditions.md) for 1 turn is added directly to target.`condition`
        - basevalue is clamped from 1 to 99
    - Otherwise, if the target has the `Toople` [condition](../Actors%20states/Conditions.md):
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
- If the target had the `Flipped` [condition](../Actors%20states/Conditions.md) at the start of this procedure, the standard damage calculation procedure is performed (see the section below for details). Otherwise, this procedure ends

### Property is anything else (standard damage calculation)
This procedure happens if the property is none of the ones mentioned above, but it's also processed in addition to the status infliction property's logic and it may be processed with the `Flip` logic.

#### DefaultDamageCalc 
The first thing that happens is [DefaultDamageCalc](DefaultDamageCalc.md) is called with the corresponding values for the parameters, with def being target.`def`.

NOTE: target.`def` does not report the effective defense here. It only reports the fixed base one. This implies many calculation bugs. TODO: study this more thoroughly to know exactly what breaks.

As for the pierce value, it's only true of all of the following are true (it's false otherwise):

- The target is an enemy party member
- The target doesn't have `AntiPierce` in its `weakness`
- The property is `Atleast1pierce` or `Pierce`

#### Magic weakness processing
After, a magic weakness hit test is performed. It succeeds if the target has `Magic` in its `weakness` as well as at least one of the following being true:

- The property is `Magic`
- The attacker is a player party member using the `Icefall`, `FrigidCoffin` or `IceRain` skill

If this test succeeds:

- basevalue is incremented
- If `commandsuccess` is true (the player suceeded the action command), weaknesshit is set to true

## Air attack logic
This section only happens if all of the following is true:

- target.battleentity.`height` is above 0.0
- target.`cantfall` is false
- target.`position` is `Flying`
- target.`lockposition` is false
- There are `NoFall` overrides

What happens here depends on if the target can be toppled which means all of the following are true:

- The target doesn't already have have the `Topple` [conditions](../Actors%20states/Conditions.md)
- The target doesn't have the `Sleep`, `Freeze` or `Numb` [conditions](../Actors%20states/Conditions.md)
- The target has `ToppleFirst` or `ToppleAirOnly` in its `weakness` 

### Toppling
If the above conditions are fufilled, the `Topple` [condition](../Actors%20states/Conditions.md) is added directly to target.`condition` array for 1 turn followed by target.battleentity.`basestate` set to 21 (`Woobly`).

### Fall
Otherwise, it means the attack happened to a toppled flying enemy or the enemy wasn't topplable and was hit while flying meaning it should be dropped to the ground. This is how this happens:

- [RemoveCondition](../Actors%20states/Conditions%20methods/RemoveCondition.md) is called with the target to remove its `Topple` [condition](../Actors%20states/Conditions.md)
- target.battleentity.`bobrange` and `bobspeed` are set to 0.0
- If target.`animid` isn't 208 (this [enemy](../../Enums%20and%20IDs/Enemies.md) id doesn't exist TODO: ???), target.battleentity.`basestate` is set to 0 (`Idle`)
- If target.`eventonfall` is above -1 (it's defined):
    - `calleventnext` is set to target.`eventonfall`
    - target.battleentity.`basestate` is set to 11 (`Hurt`)
- Otherwise (target.`eventonfall` isn't defined):
    - target.battleentity.`droproutine` is set to a new [Drop](../../Entities/EntityControl/EntityControl%20Methods.md#drop) call on target.battleentity

## Stats modifiers [conditions](../Actors%20states/Conditions.md)
This section happens as long as property isn't `NoExceptions`.

There are 2 parts to this: target's defense and attacker's attack (if one exists).

### Target's defense
If the target has the `DefenseUp` [condition](../Actors%20states/Conditions.md), it will be processed if property is null or that both of the following are true:

- property isn't `Flip`
- At least one of the following is false (piercing doesn't apply):
    - The target is an enemy party member
    - The target doesn't have `AntiPierce` in its `weakness`
    - The property is `Atleast1pierce` or `Pierce`

If the above conditions are fufilled, basevalue is decremented.

Next, if the target has the `DefenseDown` [condition](../Actors%20states/Conditions.md), it will be processed if all of the following conditions are true:
- One of the following is true:
    - [GetDefense](GetDefense.md) returns above 0 (this is target.`def` or target.`harddef`, but not the actual defense)
    - A `DefenseUp` was just processed
    - The target is a player party member while HardMode returns true meaning one of the following is true:
        - The `DoublePain` [medal](../../Enums%20and%20IDs/Medal.md) is equipped
        - [flags](../../Flags%20arrays/flags.md) 614 is true (HARDEST is active)
        - [flags](../../Flags%20arrays/flags.md) 166 is true (EX mode is active on the B.O.S.S system)
- At least one of the following is false (piercing doesn't apply):
    - The target is an enemy party member
    - The target doesn't have `AntiPierce` in its `weakness`
    - The property is `Atleast1pierce` or `Pierce`

If the above conditions are fufilled, `basevalue` is incremented.

### Attacker's attack
This part only happens if an attacker exists.

If the attacker has the `AttackUp` [condition](../Actors%20states/Conditions.md), basevalue is incremented.

If the attacker has the `AttackDown` [condition](../Actors%20states/Conditions.md), basevalue is decremented.

## `DoublePainReal` [medal](../../Enums%20and%20IDs/Medal.md) damage boost
If the target is a player party member and the `DoublePainReal` [medal](../../Enums%20and%20IDs/Medal.md) is equipped, basevalue gets multiplied by 1.25 floored then increased by 1.

## `AntlionJaws` [medal](../../Enums%20and%20IDs/Medal.md) bonus
This section happens if all of the folloiwing are true:

- The target is an enemy party member
- `AntlionJaws` [medal](../../Enums%20and%20IDs/Medal.md) is equipped on `playerdata[currentturn].trueid`
- `actionid` is 0 or below TODO: why include 0 ???
- `currentaction` is `Attack`
- Either the target has the `DefenseUp` [condition](../Actors%20states/Conditions.md) or [GetDefense](GetDefense.md) on the target is above 0 (this is target.`def` or target.`harddef`, but not the actual defense)
- At least one of the following is false (piercing doesn't apply):
    - The target is an enemy party member
    - The target doesn't have `AntiPierce` in its `weakness`
    - The property is `Atleast1pierce` or `Pierce`

If all of the above are fufilled, basevalue is increased by a certain amount. This amount is the lowest between the amount of `AntlionJaws` equipped on the target and the target's [GetDefense](GetDefense.md)  (this is target.`def` or target.`harddef`, but not the actual defense). This means that each `AntlionJaws` undoes 1 base defense of the target.

However, if target has the `DefenseUp` [condition](../Actors%20states/Conditions.md), it only counts if GetDefense is 0. This means if GetDefense is above 0 AND the target has `DefenseUp`, the condition won't count. Specifically, if the target has 1 base defense while having `DefenseUp`, at most 1 `AntLionJaws` will count (even if 2 are equipped).

? defense is not supported (no `AntLionJaws` will be processed).

## `DefDownOnFlyHard` processing
This section happens if all of the following are true:

- The target has `DefDownOnFlyHard` in its `weakness`
- target.`position` is `Flying`
- HardMode returns true meaning one of the following is true:
    - The `DoublePain` [medal](../../Enums%20and%20IDs/Medal.md) is equipped
    - [flags](../../Flags%20arrays/flags.md) 614 is true (HARDEST is active)
    - [flags](../../Flags%20arrays/flags.md) 166 is true (EX mode is active on the B.O.S.S system)

If the above conditions are fufilled, basevalue is incremented.

## `Sturdy` [condition](../Actors%20states/Conditions.md) defense bonus
If the target has the `Sturdy` [condition](../Actors%20states/Conditions.md), basevalue is decreased by 3.

## `Reflection` [condition](../Actors%20states/Conditions.md) defense bonus
If the target is a player party member with a `Reflection` [condition](../Actors%20states/Conditions.md), basevalue is decreased by the amount of `Reflection` [medal](../../Enums%20and%20IDs/Medal.md) equipped on the target.

## `LimitX10` logic
TODO: what is this??? is it even used?

- If the target doesn't have `LimitX10` in its `weakness` while property is `Atleast1` or `Atleast1pierce`:
    - basevalue is clamped from 1 to 99
- Otherwise, if the target has `LimitX10` in its `weakness`:
    - If the property is `Atleast1` or `Atleast1pierce`, basevalue is decreased by 10 TODO: why???
    - basevalue is set to 0 if it isn't higher than 1 and if it is, it's divided by 10.0 ceiled TODO: why???

## Cooldowns reset
This section allways happen.

- `caninputcooldown` is set to 0.0
- `blockcooldown` is set to 0.0

## `FrostBite` [medal](../../Enums%20and%20IDs/Medal.md) counter
If a `FrostBite` [medal](../../Enums%20and%20IDs/Medal.md) was processed to the attacker (meaning it got a `Freeze` [delayed condition](../Actors%20states/Delayed%20condition.md)), this is where the counter logic occurs.

The logic is performed by doing the following on the attacker (already determined to be an enemy party member):

- [ShowDamageCounter](../Visual%20rendering/ShowDamageCounter.md) is called with type 0 (damage) with the basevalue as the ammount starting from the world [CenterPos](../Actors%20states/CenterPos.md) of attacker and ending at Vector3.Up.
- The attacker's `hp` is decreased by basevalue clamped from 1 to its `maxhp` (meaning this can't be lethal)
- 0 is returned as the final damage value (this is because the calculations are overriden to deal no damages to the target, but they were dealt manually just now to the attacker)

## Final clamping of basevalue and return
If the target is a player party member while block is false, the final return value is basevalue clamped from 1 to 99.

Otherwise, it's basevalue clamped from 0 to 99.
