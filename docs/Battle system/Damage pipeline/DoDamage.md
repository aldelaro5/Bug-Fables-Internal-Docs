# DoDamage
DoDamage is the entrypoint of the damage pipeline. Any damage must go through it in order to be properly calculated and processed with visual effects and counters.

## Entry point
There are 10 overloads, but all of them ends up at a particular one which is the following (it will be refered to as the final overload):

```cs
private int DoDamage(MainManager.BattleData? attacker, ref MainManager.BattleData target, int damageammount, AttackProperty? property, DamageOverride[] overrides, bool block)
```
It returns the amount of damage actually dealt.

Here are the parameters:

- `attacker`: The actor dealing the damage. If null, the damage didn't come from an attacker. If it exists, it must belong to the opposite party of the `target`
- ref `target`: The actor receiving the damage. Passed as ref to allow modifying the actor's data. If the `attacker` exists, it must belong to its opposite party
- `damageammount`: The base amount of damage to deal. This number can be heavilly changed by the damage pipeline, it is only the base one to deal
- `property`: The [property](AttackProperty.md) of the attack
- `overrides`: The [overrides](DamageOverride.md) to apply to the damage pipeline. May be null or an empty array to have no overrides
- `block`: For an enemy attack, whether or not the player sucessfully blocked it (this is intended to receive `commandsuccess` as it was determined by [GetBlock](../Battle%20flow/GetBlock.md#getblock)). It doesn't matter if it's a regular block or a super block, this parameter is to signal a block of any kind as the damage pipeline will process what to do with it

### General purpose overloads
The following are general purpose overloads which just ends up calling the final one leaving some parameters to default.

(1) Calls the final overload with null/false as the other parameters (no attacker, no property, no overrides and no block)
```cs
private int DoDamage(ref MainManager.BattleData target, int ammount)
```

(2) Calls the final overload with the other parameters being null (No attacker, no property and no overrides)
```cs
private int DoDamage(ref MainManager.BattleData target, int ammount, bool block)
```

(3) Calls the final overload with the parameters sent leaving overrides to an empty array and block to false
```cs
private int DoDamage(MainManager.BattleData? attacker, ref MainManager.BattleData target, int damageammount, AttackProperty? property)
```

(4) Calls (5) with the parameters sent leaving attacker to null (meaning no attacker and no overrides)
```cs
private int DoDamage(ref MainManager.BattleData target, int damage, AttackProperty? property)
```

(5) Calls the final overload with the parameters sent leaving overrides to an empty array
```cs
private int DoDamage(MainManager.BattleData? attacker, ref MainManager.BattleData target, int damageammount, AttackProperty? property, bool block)
```

### Enemy attack overloads
The following overloads are specifically made for an enemy party member attacking a player party member.

(6) Calls the final overload with the attacker being `enemydata[enemyid]`, the target being `playerdata[playertarget]`, overrides being null and the rest being the same as the parameters sent
```cs
private int DoDamage(int enemyid, int playertarget, int damage, AttackProperty? property, bool block)
```

(7) Calls the final overload with the attacker being `enemydata[enemyid]`, the target being `playerdata[playertarget]` and the rest being the same as the parameters sent
```cs
private int DoDamage(int enemyid, int playertarget, int damage, AttackProperty? property, DamageOverride[] overrides, bool block)
```

### Player attack overloads
The following overloads are specifically made for a player party member attacking an enemy party member.

(8) Calls (9) with the parameter sent and enemytarget being the BattleControl's `target`
```cs
private int DoDamage(int playerid, AttackProperty? property)
```

(9) Calls the final overload with the following:

- `attacker`: The player party member whose `trueid` is `playerid` (it will falback to the first player party member if none exists)
- `target`: The enemy party member in `availabletargets` whose index is `enemytarget`. An exception is thrown if no enemy party member exists at that index
- `damageammount`: The return of GetPlayerAttack for the `playerid` and `commandsuccess` which is the `atk` of the corresponding player party member if `commandsuccess` is true and if it's false, it's its `atk` clamped from -99 to 1
- `property`: The same as the parameter sent
- `overrides`: If `commandsuccess` is true, this is null and if it's false, it's an array with one element being `FailSound`
- `block`: false

```cs
private int DoDamage(int playerid, int enemytarget, AttackProperty? property)
```

## Procedure
The following is a high level overview of the damage pipeline divided by sections for each groups of effects that happens.

An actor is considered to be a player party member if its battleentity has the `Player` tag. It's assumed to be an enemy party member if it doesn't. If the attacker exists, its party is assumed to be the opposte of the target's.

### block override
There are ways that block can be overriden to false. Here are all of them (they aren't mutually exclusive, if any is true, block becomes false):

- The target has the [Freeze](../Actors%20states/BattleCondition/Freeze.md) condition
- The target has the [Sleep](../Actors%20states/BattleCondition/Sleep.md) condition
- The target has the [Numb](../Actors%20states/BattleCondition/Numb.md) condition
- The target is a player party member with the `Berserker` [medal](../../Enums%20and%20IDs/Medal.md) equipped
- This is a non super block while [flags](../../Flags%20arrays/flags.md) 15 and 615 are true (chapter 1 started after the combat tutorial while FRAMEONE is active) and `overridechallengeblock` is false

### Damage calculation
This is where the final damage amount is determined.

If the target isn't Invulnerable, the final damage amount is set to the return of [CalculateBaseDamage](CalculateBaseDamage.md) with all the parameters sent (with the overriden block if applicable and property to null if it was `None`). The weaknesshit and superguarded ref parameters are passed via locals initially set to false. 

Otherwise (the target is invulnerable), it depends if the property is `NoException`:

- If it isn't, the amount is 0
- If it is, it's the original damageammount

The target is invulnerable if any of the following is true:

- Its `plating` is true
- It has the [Shield](../Actors%20states/BattleCondition/Shield.md) condition
- It is a player party member with the [Numb](../Actors%20states/BattleCondition/Numb.md) condition and the `ShockTrooper` [medal](../../Enums%20and%20IDs/Medal.md) equipped

### `plating` expiration
If the target doesn't have the [Shield](../Actors%20states/BattleCondition/Shield.md) condition, its `plating` is set to false. This allows the `plating` to not expire when hit while shielded.

### [DamageOverride](DamageOverride.md) application
8 of the 11 available `DamageOverride` are processed here if they are present in overrides. Each are processed in order they appear in the array. The other 3 were processed by [CalculateBaseDamage](CalculateBaseDamage.md) if it was called earlier:

#### `ShowCombo`
[ShowComboMessage](../Visual%20rendering/ShowSuccessWord.md#showcombomessage) is called with target.battleentity and block as block.

#### `NoSound`
The [DoDamageAnim](../Visual%20rendering/DoDamageAnim.md) call towards the end will have nosound set to true. NOTE: this is ignored if `NoDamageAnim` or `BlockSoundOnly` is processed after

#### `BlockSoundOnly`
The [DoDamageAnim](../Visual%20rendering/DoDamageAnim.md) call towards the end will have nosound set to true if block is false. NOTE: this is ignored if `NoDamageAnim` or `NoSound` is processed after

#### `FailSound`
The [DoDamageAnim](../Visual%20rendering/DoDamageAnim.md) call towards the end will have fail set to true. NOTE: this is ignored if `NoDamageAnim` is processed after

#### `NoDamageAnim`
There will not be a [DoDamageAnim](../Visual%20rendering/DoDamageAnim.md) call towards the end of the method. NOTE: processing this overrides any `NoSound`, `BlockSoundOnly`, `FailSound` and `FakeAnim` because the call won't happen

#### `NoCounter`
There will not be a [ShowDamageCounter](../Visual%20rendering/ShowDamageCounter.md) call later

#### `FakeAnim`
The [DoDamageAnim](../Visual%20rendering/DoDamageAnim.md) call towards the end will have fakeanim set to true. NOTE: this is ignored if `NoDamageAnim` is processed after

#### `DontAwake`
Prevents the attack from removing the `Sleep` [condition](../Actors%20states/Conditions.md) on the target later even if all conditions to do so are fufilled. This is only used as part of poison and fire damages from [AdvanceTurnEntity](../Battle%20flow/AdvanceTurnEntity.md).

### Player damage processing
This section only happens if the target is a player party member.

#### Block processing
This part is applicable in 2 cases:

- block is true (this is the standard reason to process the block)
- block was overriden to false due to [flags](../../Flags%20arrays/flags.md) 615 being true (FRAMEONE is active) while `commandsuccess` was true (meaning a non super block happened, but it didn't count due to FRAMEONE).

The second case is an edge case where the only thing that happens is `hasblocked` is set to true which is only used in HardSeedVenus meaning exceptionally, a `HardSeed` can be given with that attack with a non super block even if FRAMEONE is active.

The first case is the standard one:

- `hasblocked` is set to true (allows HardSeedVenus to give a `HardSeed` to the player if it gets called later as part of the enemy action)
- If this is a non super block, [ShowComboMessage](../Visual%20rendering/ShowSuccessWord.md#showcombomessage) is called with target.battleentity as the entity and true as the block
- Otherwise (meaning this was a super block):
    - `combo` is set to 2
    - The `AtkSuccess` sound is played at 1.1 pitch and 0.6 volume if it wasn't playing already (or it was, but its time was past 0.1 seconds)
    - [ShowComboMessage](../Visual%20rendering/ShowSuccessWord.md#showcombomessage) is called with target.battleentity as the entity and false as the block
    - If the `NeedlePincer` [medal](../../Enums%20and%20IDs/Medal.md) is equipped on the target and the target's battleentity still exists in `playerdata`, the target will get healed later due to the medal

#### `SpikeBod` [medal](../../Enums%20and%20IDs/Medal.md) processing
This part only happens if all of the following are true:

- The `SpikeBod` [medal](../../Enums%20and%20IDs/Medal.md) is equipped
- target.`trueid` is 1 (The target's [animid](../../Enums%20and%20IDs/AnimIDs.md) is `Beetle`)
- block is true
- `nonphyscal` is false
- There is an attacker (assumed to be an enemy party member)

In that case, the attacker will get damaged which goes like the following:

- The `Damage0` sound is played with 0.8 pitch and 0.5 volume
- The `hp` of the enemy party member corresponding to the attacker is decreased by 1 + the amount of `SuperBlock` [medals](../../Enums%20and%20IDs/Medal.md) equipped on `Beetle` then clamped from 1 to its `maxhp` (this implies that this damage cannot be lethal)
- [ShowDamageCounter](../Visual%20rendering/ShowDamageCounter.md) is called with type 0 (damage) with the amount of damage dealt just now starting from the attacker.battleentity's position + attacker.`cursoroffset` and ending to (0.0, 1.0, 1.0)

#### `PoisonTouch` [medal](../../Enums%20and%20IDs/Medal.md) processing
This part only happens if all of the following are true:

- The `PoisonTouch` [medal](../../Enums%20and%20IDs/Medal.md) is equipped on the target
- The target has the [Poison](../Actors%20states/BattleCondition/Poison.md) condition
- There is an attacker (assumed to be an enemy party member) and its `poisonres` is less than 100 (it's not immune to [poison](../Actors%20states/BattleCondition/Poison.md) inflictions)
- `nonphyscal` is false

If all of the above are fufilled, [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called to set the [Poison](../Actors%20states/BattleCondition/Poison.md) condition on the corresponding enemy party member of the attacker for 2 turns

#### `FavoriteOne` [medal](../../Enums%20and%20IDs/Medal.md) processing
This part only happens if the `FavoriteOne` [medal](../../Enums%20and%20IDs/Medal.md) is equipped on the target.

The only thing that happens is `attackedally` is set to target.`trueid`. This will only take into effect later during [AdvanceMainTurn](../Battle%20flow/Action%20coroutines/AdvanceMainTurn.md).

#### ShakeGUI call
This part always happen as part of the player damage processing.

The only thing that happens here is a ShakeGUI coroutine is started to shake the corresponding `hud` element of the target.battleentity.[animid](../../Enums%20and%20IDs/AnimIDs.md) with a range of (0.1, 0.1, 0.0) for 15.0 frames. This coroutine will change the local position of the `hud`'s first chiled to be random between (0.0 - shake.x, 0.0 - shake.y) and (shake.x, shake.y) for the next 15.0 frames before being reset to Vector3.zero.

### Enemy damage processing
This section happens if the player damage processing didn't meaning the target is an enemy party member

#### [isdefending](../Actors%20states/Enemy%20features.md#isdefending) update
This part only happens if all of the following are true:

- target.[defenseonhit](../Actors%20states/Enemy%20features.md#defenseonhit-and-isdefending) is above 0
- The final amount of damage calculated is above 0
- property isn't `Flip`. NOTE: [CalculateBaseDamage](CalculateBaseDamage.md) can change `isdefending` when property is `Flip` so this is why it can't change here (damage calculation needs to ne able to override this logic hence why this check is done)

If all the above are fufilled, it means the enemy supports defending itself and it sustained damage which causes updates to its guarding state. When that happens, `isdefending` is set to false if the target [IsStopped](../Actors%20states/IsStopped.md) or to true if it's not.

#### [hitaction](../Actors%20states/Enemy%20features.md#hitaction) update
target.`hitaction` can change under 2 conditions (these aren't mutually exclusive, they are applied in order, but `hitaction` may not be set if neither apply):

- It's set to true if target.[defenseonhit](../Actors%20states/Enemy%20features.md#defenseonhit-and-isdefending) is -1 and its [position](../Actors%20states/BattlePosition.md) isn't `Underground`. This is only the case for an `Underling` [enemy](../../Enums%20and%20IDs/Enemies.md) under normal gameplay
- It's set to !`enemy` (false on the [enemy phase](../Battle%20flow/Main%20turn%20life%20cycle.md#enemy-phase) / [delproj](../Actors%20states/Delayed%20projectile.md#delayed-projectile) / enemy `firststrike` / enemy `hitaction`, true otherwise) if the target.[onhitaction](../Actors%20states/Enemy%20features.md#onhitaction) condition is fufilled which is:
    - If it's 1, it's always fufilled
    - If it's 2, it's only fufilled if target.`position` is `Flying`
    - If it's 3, it's only fufilled if target.`position` is `Ground`

After, every other enemy party members other than the target who has the target.`animid` (its [enemy](../../Enums%20and%20IDs/Enemies.md) id) in their [chargeonotherenemy](../Actors%20states/Enemy%20features.md#chargeonotherenemy) has their `hitaction` set to !`enemy` (false on the [enemy phase](../Battle%20flow/Main%20turn%20life%20cycle.md#enemy-phase) / [delproj](../Actors%20states/Delayed%20projectile.md#delayed-projectile) / enemy `firststrike` / enemy `hitaction`, true otherwise).

#### Undigging
This part only happens if all of the following are true:

- The final amount of damages calculated earlier is higher than 0
- target.[position](../Actors%20states/BattlePosition.md) is `Underground` 
- `mainturn` ([AdvanceMainTurn](../Battle%20flow/Action%20coroutines/AdvanceMainTurn.md)) isn't in progress

If the above conditions are fufilled, then UnDig is called with the target as ref which does the following on it:

- battleentity.`digging` is set to false
- battleentity.`overridejump` is set to true
- [Jump](../../Entities/EntityControl/EntityControl%20Methods.md#jump) is called on the battleentity with a height of 15.0
- [position](../Actors%20states/BattlePosition.md) is set to `Ground`

#### `turnsnodamage` reset
If the final amount of damages calculated earlier is higher than 0, target.`turnsnodamage` is set to -1.

### Player action command success
If `commandsuccess` is true and the attacker is a player party member:

- `wordroutine` is stopped if it was in progress (followed by the destruction of `commandword` if it was)
- `wordroutine` is set to a new [ShowSuccessWord](../Visual%20rendering/ShowSuccessWord.md) call with target.battleenetity as the t, without block and the super being the ref weaknessonhit value obtained from [CalculateBaseDamage](CalculateBaseDamage.md) done earlier (false if it wasn't called)

### target.`hp` decrease and clamping
target.`hp` gets decreased by the final amount of damage calculated earlier, but it also gets clamped after. The higher bound of the clamp is always target.`maxhp`, but the lower bound depends on some conditions.

Here they are in order (they are mutually exclusive, the first one applies):

#### Clamp from 10
This happens if all the following are true:

- The target is an enemy party member
- The target has a `SurviveWith10` weakness
- [currentaction](../Player%20UI/Pick.md) is `ItemList` (meaning this was an item use) or `lastskill` is not 9 (this attack came from any player [skills](../../Enums%20and%20IDs/Skills.md) other than `PeebleToss`)

#### Clamp from 1
This happens if the target is an enemy party member with an `AlwaysSurvive` weakness.

#### Clamp from 0
This is the default clamp that happens if the other 2 didn't.

### Player selection cycle reset
If the target is a player party member with an `hp` of 0 (meaning it just died as a result of the attack), SetLastTurns is called which resets `lastturns` to a new aray with the length being the amount of free players - 1 and all elements being -1 (this resets the player selection cycle).

### Target waking up
This section only applies if all of the following is true:

- The target had the [Sleep](../Actors%20states/BattleCondition/Sleep.md) condition at the start of the method
- The property isn't `Sleep`
- target.`hp` is above 0
- The final damage amount calculated earlier is above 0. NOTE: [CalculateBaseDamage](CalculateBaseDamage.md) may override this if the property is one of the status infliction one (except `Flip`) and the infliction occured because this clause isn't part of the damage calculation. This means that if the amount is 0, but an infliction removing `Sleep` occured, the `cantmove` won't change which is a different logic than this method enforces
- There was no [DamageOverride](DamageOverride.md) of `DontAwake` processed earlier
- If the target is an player party member, the `HeavySleeper` [medal](../../Enums%20and%20IDs/Medal.md) must not be equipped

If all of the above are fufilled, the following happens:

- [RemoveCondition](../Actors%20states/Conditions%20methods/RemoveCondition.md) is called to remove the [Sleep](../Actors%20states/BattleCondition/Sleep.md) condition on the target
- target.`isasleep` is set to false
- target.`cantmove` is set to 1. NOTE: This is incorrect because it means for a player party member, that remaining actor turn will advance immeidately after the enemy phase so they get to act, but for an enemy party member, they loose their entire main turn because the actor turn won't advance after the player phase
- [UpdateAnim](../Visual%20rendering/UpdateAnim.md) is called with true as the onlyplayer

### `LifeSteal` processing
This section only happens if the attacker is a player party member with the `LifeSteal` [medal](../../Enums%20and%20IDs/Medal.md) equipped while `nolifesteal` is false.

If the above is fufilled, the following happens:

- The `Heal` sound is played
- The amount to heal is set to the final damage amount calculated earlier / 2.0 (or 4.0 if the `HPFunnel` [medal](../../Enums%20and%20IDs/Medal.md) is equipped on the attacker) then clamped from 1.0 to 99.0 then floored
- The attacker's `hp` is increased by the amount just calculated clamped from 0 to the attacker's `maxhp`
- [ShowDamageCounter](../Visual%20rendering/ShowDamageCounter.md) is called with type 1 (HP) for the amount to heal calculated earlier (without `maxhp` clamp) starting from the attacker's battleentity's position + (0.0, 1.75, 0.0) and ending at (0.0, 3.0, 0.0)
- `nolifesteal` is set to true which prevents the medal from processing until the next [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md) call

### Stat down on block properties processing
This section applies if the target.`hp` is above 0, the target doesn't have the `Shield` [condition](../Actors%20states/Conditions.md) and the property is `DefDownOnBlock` or `AtkDownOnBlock`.

This will call [StatusEffect](../Actors%20states/Conditions%20methods/StatusEffect.md) to give the target a [condition](../Actors%20states/Conditions.md) with visual effect. The amount of turn to inflict is 2 turns unless block is true where it's 1 turn instead. On top of this, it inflicts for one more turn if the target is a player party member (meaning 3 turns or 2 with block).

As for the actual condition, it's `DefenseDown` if the property is `DefDownOnBlock` and it's `AttackDown` if the property is `AtkDownOnBlock`.

### `NeedlePincer` [medal](../../Enums%20and%20IDs/Medal.md) processing
If a heal was requested earlier as a result of blocking as a player party member with a `NeedlePincer` [medal](../../Enums%20and%20IDs/Medal.md), this is where the [Heal](../Actors%20states/Heal.md) call happens with the target as the entity and the ammount being the amount of `NeedlePincer` equipped on the target.

### [DoDamageAnim](../Visual%20rendering/DoDamageAnim.md) call
Unless there was a `NoDamageAnim` damage override processed earlier, [DoDamageAnim](../Visual%20rendering/DoDamageAnim.md) is called with these parameters:

- entity: target
- damage: The final amount of damage calculated
- block: block
- noanim: true
- nosound, fail and fakeanim, they are false unless a damage override processed earlier decided otherwise

### Final processing
This section always happen at the end of the method:

- `combo` is incremented
- RefreshMusic is called which is an empty method
- `lastdamage` is set to the final amount of damage calculated earlier
- If the target is an enemy party member, `damagethisturn` is increased by the final amount of damage calculated earlier
- The final amount of damage calculated earlier is returned, ending the method
