# Enemies
[Enemies](../Enums%20and%20IDs/Enemies.md) data are split in 3 TextAssets in the game loaded on boot:

- By LoadEssentials:
    - `Ressources/data/EnemyData`
- By SetVariables:
    - `Ressources/data/TattleList`
    - `EnemyTattle` from the corresponding dialogue directory of the current [languageid](../SetText/languageid.md)

`EnemyData` is also loaded on the Start and Update of the PauseMenu.

## `EnemyData` data
The asset contains one line per [enemy](../Enums%20and%20IDs/Enemies.md) whose id corresponds to the line index. Each line contains fields separated by `@`:

|Loaded index|Name|Type|Description|
|-----------:|----|----|-----------|
|0|battleentity.animid|int|The [AnimID](../Enums%20and%20IDs/AnimIDs.md) of the entity bound to this enemy (can be overriden if it's a variant, see the section below about it for details)|
|1|hp|int|The base max HP of the enemy (can be influenced by difficulty scaling, see the section below about it for details)|
|2|def|int|The base defense of the enemy (can be influenced by difficulty scaling, see the section below about it for details)|
|3|exp|int|The base amount of exploration points this enemy gives (can be overriden to 0 or be scaled by the instance.`partylevel`, see the section below about it for details)|
|4|money|int|The base amount of berries this enemy drops|
|5|cursoroffset.x|float|The x component of the offset to apply to the selection cursor upon targeting the enemy|
|6|cursoroffset.y|float|The y component of the offset to apply to the selection cursor upon targeting the enemy|
|7|cursoroffset.z|float|The z component of the offset to apply to the selection cursor upon targeting the enemy|
|8|poisonres|int|The base [poison](../Battle%20system/Actors%20states/BattleCondition/Poison.md) resistance of the enemy|
|9|freezeres|int|The base [freeze](../Battle%20system/Actors%20states/BattleCondition/Freeze.md) resistance of the enemy|
|10|numbres|int|The base [numb](../Battle%20system/Actors%20states/BattleCondition/Numb.md) resistance of the enemy|
|11|sleepres|int|The base [sleep](../Battle%20system/Actors%20states/BattleCondition/Sleep.md) resistance of the enemy|
|12|size|float|The width of the enemy used for calculating positioning|
|13|battleentity.freezesize.x|float|The x component of the ice cube size when the entity is frozen|
|14|battleentity.freezesize.y|float|The y component of the ice cube size when the entity is frozen|
|15|battleentity.freezesize.z|float|The z component of the ice cube size when the entity is frozen|
|16|battleentity.freezeoffset.x|float|The x component of the ice cube offset from the entity's center when the entity is frozen|
|17|battleentity.freezeoffset.y|float|The y component of the ice cube offset from the entity's center when the entity is frozen|
|18|battleentity.freezeoffset.z|float|The z component of the ice cube offset from the entity's center when the entity is frozen|
|19|[position](../Battle%20system/Actors%20states/BattlePosition.md)|`BattlePosition` (string or int)|The starting battle position of the enemy on the battle field (`Random` has special logic, see the special fields logic section below for details)|
|20|battleentity.height|float|The `height` of the entity, If the [position](../Battle%20system/Actors%20states/BattlePosition.md) is `Flying` and the value is below 2.0, it is overriden to 2.0|
|21|battleentity.bobspeed|float|The `bobspeed` of the entity|
|22|battleentity.bobrange|float|The `bobrange` of the entity|
|23|[weakness](../Battle%20system/Actors%20states/Enemy%20features.md#weakness)|`{` separated list of AttackProperty (int or string)|The list of [AttackProperty](../Battle%20system/Damage%20pipeline/AttackProperty.md) that applies to this enemy|
|24|[weight](../Battle%20system/Actors%20states/Enemy%20features.md#weight)|float|A visual modifier that controls if and how much the enemy can get launched if attacked, should be between 0.0 (furthest launch) and 100.0 (no launch)|
|25|Base [Enemy](../Enums%20and%20IDs/Enemies.md) id (loaded as animid)|int|The [Enemy](../Enums%20and%20IDs/Enemies.md) id that this is a variant of (see the section below about enemy variant for details), can be omitted if it's a negative number|
|26|[eventondeath](../Battle%20system/Actors%20states/Enemy%20features.md#eventondeath)|int|The [EventDialogue](../Battle%20system/Battle%20flow/EventDialogue.md) to trigger when the enemy dies as detected by [CheckDead](../Battle%20system/Battle%20flow/Action%20coroutines/CheckDead.md), can be ommited if it's -1|
|27|[moves](../Battle%20system/Actors%20states/Enemy%20features.md#moves)|int|The amount of actor turn the enemy has normally each main turn during the [enemy phase](../Battle%20system/Battle%20flow/Main%20turn%20life%20cycle.md#enemy-phase)|
|28|[notaunt](../Battle%20system/Actors%20states/Enemy%20features.md#notaunt)|bool|Tells if the enemy can't receive the [Taunted](../Battle%20system/Actors%20states/BattleCondition/Taunted.md) condition by using the `Taunt` [skill](../Enums%20and%20IDs/Skills.md)|
|29|[cantfall](../Battle%20system/Actors%20states/Enemy%20features.md#cantfall)|bool|Tells if the enemy cannot drop or have its [position](../Battle%20system/Actors%20states/BattlePosition.md) set to `Ground`|
|30|fixedexp|bool|Tells if the exp field should not be scaled and is a fixed number (see the section below about EXP for details)|
|31|[notired](../Battle%20system/Actors%20states/Enemy%20features.md#notired)|bool|Tells if exhaution increases won't happen if they were set to happen normally (with caveats, check the documentation of the feature to learn more)|
|32|[hidehp](../Battle%20system/Actors%20states/Enemy%20features.md#hidehp)|bool|Tells if the `hp` and `def` stats of the enemy should not be shown even after spying|
|33|[deathtype](../Battle%20system/Actors%20states/Enemy%20features.md#deathtype)|int|The manner in which the enemy dies (This isn't a `DeathType`, more on the feature's documentation)|
|34|[chargeonotherenemy](../Battle%20system/Actors%20states/Enemy%20features.md#chargeonotherenemy)|`;` separated list of int or `-1` or empty|The list of [enemy](../Enums%20and%20IDs/Enemies.md) ids that will cause this enemy to perform a [hitaction](../Battle%20system/Actors%20states/Enemy%20features.md#hitaction) when any enemy id in that list takes damages in the player phase. See the feature's documentation to learn more. This field also has special loading logic, check the section below for more details|
|35|hardatk|int|The amount to increase the attack on Hard Mode or EX (see the section below about difficulty scaling for details). Not loaded if the hard difficulty scaling doesn't apply and the value remains at 0|
|36|<No field>|int|The base amount to increase the max HP on Hard Mode, EX or HARDEST (this is NOT `hardhp` which stays at 0 and is UNUSED). See the section below about difficulty scaling for details|
|37|<No field>|int|The base amount to increase the defense on Hard Mode, EX or HARDEST (this is NOT `hardef` which stays at 0 and is UNUSED). See the section below about difficulty scaling for details|
|38|[defenseonhit](../Battle%20system/Actors%20states/Enemy%20features.md#defenseonhit-and-isdefending)|int|The amount of point of defenses granted to the enemy when `isdefending` is true (check the feature's documentation to learn more)|
|39|itemoffset.x|float|The x component of the offset to render an item relative to the enemy|
|40|itemoffset.y|float|The y component of the offset to render an item relative to the enemy|
|41|itemoffset.z|float|The z component of the offset to render an item relative to the enemy|
|42|<No field>|bool|If true, the battleentity.`basestate` is set to 13 (`BattleIdle`)|
|43|Portrait sprite index|int|Index of the sprite in `Sprites/Items/EnemyPortraits` (if this is negative, this defaults to the enemy id, see the GetEnemyPortrait section below about this for more details)|
|44|[notattle](../Battle%20system/Actors%20states/Enemy%20features.md#notattle)|bool|Tells if the enemy is not spy-able. Check the feature's documentation for more details. This fields also has special loading logic that can override it to true, check the section below for more detials|
|45|[eventonfall](../Battle%20system/Actors%20states/Enemy%20features.md#eventonfall)|int|The [EventDialogue](../Battle%20system/Battle%20flow/EventDialogue.md) to trigger when the enemy drops, can be ommited if it's -1|
|46|[onhitaction](../Battle%20system/Actors%20states/Enemy%20features.md#onhitaction)|int|If 1, the enemy will process a [hitaction](../Battle%20system/Actors%20states/Enemy%20features.md#hitaction) when hit. If 2, it will only when its [position](../Battle%20system/Actors%20states/BattlePosition.md) is `Flying` and if 3, it will only when its `position` is `Ground`|
|47|[actimmobile](../Battle%20system/Actors%20states/Enemy%20features.md#actimmobile)|bool|Prevent `cantmove` to be anything higher than 0 or [IsStopped](../Battle%20system/Actors%20states/IsStopped.md) to return true as a result of being inflicted with a stopping [condition](../Battle%20system/Actors%20states/Conditions.md). This works for most, but not all cases, check the feature's documentation to learn more|
|48|[sizeonfreeze](../Battle%20system/Actors%20states/Enemy%20features.md#sizeonfreeze)|float|The size of the enemy on freeze. If the value is less than 0.1, it is be overriden to `size` + 0.25, check the feature's documentation to learn more|

The data will be loaded by into `enemydata[id, x]`, where `id` is the [enemy](../Enums%20and%20IDs/Enemies.md) id and `x` is the loaded index. On the pause menu, every lines is loaded into `enemydata` all at once, but the meanings do not change.

### GetEnemyData
While the data loading happens in `LoadEssentials`, the actual parsing and usage of it is done by another method called `GetEnemyData`. It receives an [enemy](../Enums%20and%20IDs/Enemies.md) to obtain the data, a bool called createentity that tells whether or not the `battleentity` should be created too and a bool called noexp that disallows any positive numbers to be set to `exp`. This returns a `BattleData` containing all the informations of the enemy. 

```cs
public static BattleData GetEnemyData(int id, bool createentity, bool noexp)
```

All `battleentity` fields are loaded only when the createentity parameter is true. When that happens, `battleentity` will be created via [CreateNewEntity](../Entities/EntityControl/EntityControl%20Methods.md#createnewentity) with name `enemyX` where X is the enemy id with dummy animid and position (animid 0 which is `Bee` and position (0.0, -10.0, 0.0)) as those can be overriden later.

#### Enemy variant
The base [Enemy](../Enums%20and%20IDs/Enemies.md) id field uses a feature to redirect most of the fields loaded to be the ones belonging to another enemy entirely. It is used to add an enemy variant which only shares some fields with its base enemy, but it has its own id. It also has the effect of overriden the `animid` field to be the base enemy id instead. Here are the list of fields that are shared:

- `eventondeath`
- `battleentity.height`
- `battleentity.bobspeed`
- `battleentity.bobrange`
- `moves`
- `notaunt`
- `cantfall`
- `notired`
- `fixedexp`
- `position`
  
There are however some fields that are excluded from this if the createentity is true and the original [Enemies](../Enums%20and%20IDs/Enemies.md) id is `FireKrawler`, `FireWarden`, `FireCape`, `IceKrawler` or `IceWarden`. They will load the following fields from their base entry (`Krawler`, `Warden` or `CursedSkull`) instead on top of the standard variant system:

- `moves`
- `notaunt`
- `cantfall`
- `notired`
- `fixedexp`
- `position`

Every other loading logic or fields will be loaded as if it was the enemy of the corresponding base [Enemy](../Enums%20and%20IDs/Enemies.md) id. This imply that every other field of the actual enemy will be ignored and they can be left as dummy values.

#### BattleEntity special initialisation
When createentity is true, on top of actually creating the entity and setting it to the `battleentity` field, the entity itself has special logic when it comes to initialise it. This section will only mention anything that's outside the scope of loading a data field:

- `battleentity.overridejump` is set to true
- `battleentity.battle` is set to true
- `battleentity.alwaysactive` is set to true
- `battleentity.onground` is set to true
- If this enemy hasn't been seen while the map is `CaveOfTrials` or it's a `TANGYBUG` or the enemy is `FireKrawler`, `FireCape` or `FireWarden` while [flag](../Flags%20arrays/flags.md) 664 is false (not yet approached the oven during Chapter 7), the following happens (this can have an impact on the `exp` field of the `BattleData`, see the section below for more details):
    - `battleentity.name` gets prepended with the `COT` [Modifier](../Entities/EntityControl/Modifiers.md)
    - `battleentity.hologram` is set to true
    - `battleentity.cotunknown` is set to true
    - RefreshCOT is invoked in 0.1 seconds on the `battleentity`
- `battleentity.tag` is set to `Enemy`
- `battleentity.gameObject.layer` is set to 9 (`Follower`)
- `entity` is set to the `battleentity`
- If the `position` is `Flying` and `battleentity.height` is below 2.0, `battleentity.height` is overriden to 2.0
- `battleentity.initialheight` is set to `battleentity.height`
- CreateHPBar is called on the entity
- `battleentity.emoticonoffset` is set to the `cursoroffset`
- If the `position` is `Underground`:
    - InstantDig is called on the `battleentity`
    - `battleentity.height` and `battleentity.initialheight` are set to 0.0
- If the field 42 of `enemydata` is true, `battleentity.basestate` is set to 13 (`BattleIdle`)

After this, the `entity.destroytype` gets mapped to a DeathType depending on the value of the `BattleData`'s [deathtype](../Battle%20system/Actors%20states/Enemy%20features.md#deathtype) (this also changes the `battleentity`'s because they hold the same reference).

#### Exp logic
The `exp` field has particularily complex logic to determine its value:

- It is always 0 if the sent noexp value is true
- It is left at the default value of 0 if instance.`partylevel` is at least 27 (it's maxed) or if [flag](../Flags%20arrays/flags.md) 613 is true (RUIGEE is active)
- If the `fixedexp` field is true, then the amount is always the raw one coming from `enemydata` with no modifications to it
- If none of the cases above applied, the result is the return of MainManager.GetEXP with the `exp` field from `enemydata`, the instance.`partylevel` and the enemy id (the base id if it's a variant)

The return of MainManager.GetEXP implies even more logic where the base `exp` field can be changed. Here is the process used to calculate the final value:

1. The base EXP amount is determined. It's the same than the sent exp value unless the enemy is a `WaspTrooper` or `WaspHealer` which can increase this value:
    - If the current [map](../Enums%20and%20IDs/Maps.md) is `MetalLake` or the [area](../Enums%20and%20IDs/librarystuff/Areas.md) is `WaspKingdom`, the value is multiplied by 1.65 and then ceiled
    - Otherwise, if the current [area](../Enums%20and%20IDs/librarystuff/Areas.md) is `RubberPrison`, the value is multiplied by 2.5 ceiled. On top of this, if it's a `WaspHealer`, 10 is added to the amount
2. The value gets multiplied by map.`expmulti` which is a field that must be defined on the map's prefab. The result isn't rounded yet
3. The value gets subtracted by (level - 1) * 2.5 and the result is clamped from 1.0 to 99.0 then floored
5. The now integer value is decremented and returned

Additionally, GetEnemyData can also change yet again the value after it was calculated in 2 special cases (they both can happen and stack and they can happen regardless of how the initial `exp` value was determined):

1. [flag](../Flags%20arrays/flags.md) 162 is true (during a B.O.S.S or Cave Of Trials session): the value gets multiplied by 0.2, floored and then clamped from 0 to 5
2. createentity is true, [flag](../Flags%20arrays/flags.md) 166 is false (EX mode isn't active on the B.O.S.S. system) and the `battleentity` is a `hologram` (see the section above about the `battleentity` initialisation for when this happens): the value is multiplied by 0.1 and floored

Then, [StartBattle](../Battle%20system/StartBattle.md) can further change the `exp` If all of the following conditions are true:

- The [enemy](../Enums%20and%20IDs/Enemies.md) is a `Krawler`, `CursedSkull` or `Cape`
- battleentity.`forcefire` is true or we are in the `GiantLair` [area](../Enums%20and%20IDs/librarystuff/Areas.md) except for the `GiantLairFridgeInside` [map](../Enums%20and%20IDs/Maps.md)
- instance.`partylevel` is less than 27 (meaning it's not maxed)
- [flags](../Flags%20arrays/flags.md) 613 is false (RUIGEE is inactive)

If that happens, the `exp` is incremented by the floored result of a lerp from 10.0 to 3.0 with a factor of instance.`partylevel` / 27.0

Finally, while all of the above describes the `exp` field itself, [CheckDead](../Battle%20system/Battle%20flow/Action%20coroutines/CheckDead.md) calls BattleControl.GetEXP (not to be confused with MainManager.GetEXP) to determine the rewarded exp amount. This method contains further special logic where it receives the `exp` and `fixedexp` fields of the enemy as well as its enemy id. Here's what the method does:

- If [flags](../Flags%20arrays/flags.md) 613 is true (RUIGEE is active) or `partylevel` is 27 (max rank), the return is 0 (effectively disables all exp gains to the player)
- Otherwise:
    - If the `DoublePain` [medal](../Enums%20and%20IDs/Medal.md) is equipped or [flags](../Flags%20arrays/flags.md) 614 is true (HARDEST is active), `exp` increases by (`exp` * 0.15 ceiled which is basically 15% ceiled)
    - If the `EXPBoost` [medal](../Enums%20and%20IDs/Medal.md) is equipped, `exp` increases by (`exp` * 0.50 ceiled which is basically 50% ceiled)
    - A result is determined from there (only the first one that applies is returned):
        - If [Flags](../Flags%20arrays/flags.md) 166 is true (during a B.O.S.S session in EX mode), the return is `exp` clamped from 0 to 5
        - Otherwise, if `fixedexp` is true, `exp` is returned without any clamping
        - Otherwise, if the enemy is among [ChomperBrute](../Battle%20system/Enemy%20actions/Enemies/ChomperBrute.md), [ToeBitter](../Battle%20system/Enemy%20actions/Enemies/ToeBiter.md), [DeadLanderA](../Battle%20system/Enemy%20actions/Enemies/DeadLanderA.md), [DeadLanderB](../Battle%20system/Enemy%20actions/Enemies/DeadLanderB.md) or [DeadLanderG](../Battle%20system/Enemy%20actions/Enemies/DeadLanderG.md), the return is `exp` clamped from 0 to 20
        - Otherwise (neither of the above applied), the return is clamped from 0 to 15

#### Special fields logic
Additionally, some [BattleData](../Battle%20system/Actors%20states/BattleData.md) fields have special logic attached to them:

- `entityname`: See the section about `EnemyTattle` below
- `holditem`: always set to -1
- `chargeonotherenemy`: The value depends on the loaded data:
    - If it's `-1`, the value is an empty array
    - If it's empty, the value is null
    - If it's a `;` separated list of [enemy](../Enums%20and%20IDs/Enemies.md) ids, each element will be converted to int and the value will be that array, but for any element that's empty in that list, the element will be -1 instead
- `notattle`: If the loaded data value is false, it will still be overriden to true if any of these is true:
    - This enemy hasn't been seen while the [map](../Enums%20and%20IDs/Maps.md) is `CaveOfTrials`
    - createentity is true and this is a `FireKrawler`, `FireCape` or `FireWarden` [enemy](../Enums%20and%20IDs/Enemies.md) while [flag](../Flags%20arrays/flags.md) 664 is false (not yet approached the oven during Chapter 7)
- `sizeonfreeze` If the loaded value is less than 0.1, it will be overriden to `size` + 0.25
- `hardatk`, `hp` and `def`: See the section below detailing how enemy stats difficulty scaling works
- `maxhp`: Always set to `hp` after it was calculated post difficulty scaling
- `position`: If its value is Random, then it will be determined randomly and uniformly between Ground and Flying, but there's an exception if the enemy is a `Mushroom` and [flag](../Flags%20arrays/flags.md) 24 is true (received the Turn Relay tutorial) in which case, the position field is Flying
- `entity`: If createentity is true, this is set to the `battleenentity` being created
- `condition`: Always set to a new list
- `cantmove`: Always set to -`moves` + 1 which gives it `moves` amount of actions available
- `harddef` and `hardhp`: These fields are practicaly unused because they always have a value of 0 which doesn't have any impact in the game

#### Stats difficulty scaling
There are some fields in enemy data that are influenced by the different difficulty tiers the game has. This only happens if any of the following is true:

- The `DoublePain` [Medal](../Enums%20and%20IDs/Medal.md) is equipped
- [Flags](../Flags%20arrays/flags.md) 614 is true (HARDEST is active)
- [Flags](../Flags%20arrays/flags.md) 166 is true (EX mode is active on the B.O.S.S. system)  

If the above is met, it will always have the following effect:

- `hardatk`: Set to the loaded data value (this is left at 0 if hard scaling didn't apply in the first place)
- `hp`: Increased by `enemydata` field 36
- `def`: Increased by `enemydata` field 37

On top of this, if HARDEST was active, the following applies after:

- `hardatk`: Gets incremented
- `hp`: Gets increased by 15% then ceiled
- `def`: If [flags](../Flags%20arrays/flags.md) 300 is true (Chapter 4 started) and the existing `def` wasn't negative, it gets incremented

On top of this, if EX mode is active on the B.O.S.S. system, then there are additional effects after everything if the enemy isn't `WaspGeneral`, `KeyR`, `KeyL` or `Tablet`:

- `hardatk`: Gets increased by 15% ceiled
- `hp`: Gets increased by 15% ceiled. If the result ends up above 90, it is then decreased by 15% ceiled.
- `def`: Gets clamped from 1 to 99

### GetEnemyPortrait
This field is loaded, but never written to. Its only usage is in the GetEnemyPortrait method which by default returns the value directly (or the enemy id if the value is negative).

However, the return can be overridden to 225 for a `Cape`, 226 for a `Krawler` and 227 for a `CursedSkull` if [flag](../Flags%20arrays/flags.md) 664 is true (approached the oven during Chapter 7). This overrides the portrait sprite to include the fire variants of the [enemy](../Enums%20and%20IDs/Enemies.md) under this condition.

## `EnemyTattle` data
The asset contains one line per [Enemy](../Enums%20and%20IDs/Enemies.md) whose id corresponds to the line index. Each line contains fields separated by `@`:

|Loaded index|Name|Type|Description|
|------------:|----|----|-----------|
|0|Name|[SetText](../SetText/SetText.md) string|The name of the enemy|
|1|Biography|[SetText](../SetText/SetText.md) string|The biography of the enemy|
|2|Vi's spy|[SetText](../SetText/SetText.md) string|Vi's Spy dialogue|
|3|Kabbu's spy|[SetText](../SetText/SetText.md) string|Kabbu's Spy dialogue|
|4|Leif's spy|[SetText](../SetText/SetText.md) string|Leif's Spy dialogue|

The data will be loaded into `librarydata[1, id, x]` where `id` is the [Enemy](../Enums%20and%20IDs/Enemies.md) id and `x` is the loaded index. The name is also loaded into `enemynames`.

Enemies not meant to have a [Bestiary entry](../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md) by convention have only the name field defined and the rest are dummy fields. The biography's dummy value is typically "biotattle" as well as the spy text fields being "beetattle", "beetletattle" and "mothtattle" respectively.

During `GetEnemyData`, the typical `entityname` value is the one loaded into `enemynames` but it will be menutext 59 (?????) during Cave Of Trials for an unseen enemy or if createentity is true and the enemy is `FireKrawler`, `FireCape` or `FireWarden` while [flag](../Flags%20arrays/flags.md) 664 is false (not yet approached the oven during Chapter 7).

## `TattleList` data
The asset contains the order to render the [Bestiary entry](../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md) in the [Library List Type](../ItemList/List%20Types%20Group%20Details/Library%20List%20Type.md) where each line contains one field:

|Name|Type|Description|
|----|----|-----------|
|Enemy id|int|The [Enemies](../Enums%20and%20IDs/Enemies.md) id of the enemy to render at the position of the line index|

The data will be loaded into `libraryorder[1, i]` where i is the line index.
