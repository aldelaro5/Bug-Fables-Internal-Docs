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
|8|poisonres|int|The poison resistance of the enemy|
|9|freezeres|int|The freeze resistance of the enemy|
|10|numbres|int|The numb resistance of the enemy|
|11|sleepres|int|The sleep resistance of the enemy|
|12|size|float|The width of the enemy used for calculating positioning|
|13|battleentity.freezesize.x|float|The x component of the ice cube size when the entity is frozen|
|14|battleentity.freezesize.y|float|The y component of the ice cube size when the entity is frozen|
|15|battleentity.freezesize.z|float|The z component of the ice cube size when the entity is frozen|
|16|battleentity.freezeoffset.x|float|The x component of the ice cube offset from the entity's center when the entity is frozen|
|17|battleentity.freezeoffset.y|float|The y component of the ice cube offset from the entity's center when the entity is frozen|
|18|battleentity.freezeoffset.z|float|The z component of the ice cube offset from the entity's center when the entity is frozen|
|19|position|BattlePosition (string or int)|The battle position of the enemy on the battle field (`Random` has special logic, see the special fields logic section below for details)|
|20|battleentity.height|float|The `height` of the entity, If the `position` is `Flying` and the value is below 2.0, it is overriden to 2.0|
|21|battleentity.bobspeed|float|The `bobspeed` of the entity|
|22|battleentity.bobrange|float|The `bobrange` of the entity|
|23|weakness|`{` separated list of AttackProperty (int or string)|The list of attack properties the enemy ??? TODO: could be more than weaknesses|
|24|weight|float|???|
|25|Base [Enemy](../Enums%20and%20IDs/Enemies.md) id (loaded as animid)|int|The [Enemy](../Enums%20and%20IDs/Enemies.md) id that this is a variant of (see the section below about enemy variant for details), can be omitted by putting any negative number|
|26|eventondeath|int|The EventDialogue to trigger when the enemy dies ???|
|27|moves|int|The amount of actions the enemy has normally each turn during the enemy phase|
|28|notaunt|bool|Tells if the enemy can't be taunted ???|
|29|cantfall|bool|Tells if the enemy cannot fall ???|
|30|fixedexp|bool|Tells if the exp field should not be scaled and is a fixed number (see the section below about EXP for details)|
|31|notired|bool|???|
|32|hidehp|bool|Tells if the HP of the enemy should always be hidden even after spying|
|33|deathtype|int|The manner in which the enemy dies (This isn't a DeathType, see the `battleentity` section below for details on how it is mapped) TODO: there might be more to this|
|34|chargeonotherenemy|`;` separated list of int or `-1` or empty|The list of [enemy](../Enums%20and%20IDs/Enemies.md) ids that will cause this enemy to perform a `hitaction` when any enemy id in that list takes damages in the player phase. See the special fields logic section below for more details on which format gives what value TODO: seems reasonable, but recheck to make sure|
|35|hardatk|int|The amount to increase the attack on Hard Mode or EX (see the section below about difficulty scaling for details). Not loaded if the hard difficulty scaling doesn't apply and the value remains at 0|
|36|<No field>|int|The base amount to increase the max HP on Hard Mode, EX or HARDEST (this is NOT `hardhp` which stays at 0 and is unused). See the section below about difficulty scaling for details|
|37|<No field>|int|The base amount to increase the defense on Hard Mode, EX or HARDEST (this is NOT `hardef` which stays at 0 and is unused). See the section below about difficulty scaling for details|
|38|defenseonhit|int|The amount to increase the defense on hit ???|
|39|itemoffset.x|float|The x component of the offset to render an item relative to the enemy|
|40|itemoffset.y|float|The y component of the offset to render an item relative to the enemy|
|41|itemoffset.z|float|The z component of the offset to render an item relative to the enemy|
|42|<No field>|bool|If true, the `battleentity.basestate` is set to 13 (`BattleIdle`)|
|43|Portrait sprite index|int|Index of the sprite in `Sprites/Items/EnemyPortraits` (if this is negative, this defaults to the enemy id, see the section below about this for more details)|
|44|notattle|bool|Tells if the enemy is not spy-able (this can be overriden to true, see the section about special fields logic for details)|
|45|eventonfall|int|The EventDialogue to trigger when the enemy falls ???|
|46|onhitaction|int|If 1, the enemy will process a `hitaction` when hit. If 2, it will only when its `position` is `Flying` and if 3, it will only when its `position` is `Ground`|
|47|actimmobile|bool|???|
|48|sizeonfreeze|float|The size of the enemy on freeze. If the value is less than 0.1, it is be overriden to `size` + 0.25 TODO: why is this separate than the entity one ???|

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

After this, the `entity.destroytype` gets mapped to a DeathType depending on the value of the `BattleData`'s `deathtype` (this also changes the `battleentity`'s because they hold the same reference):

|deathtype|entity.destroytype|
|--------:|------------------|
|0 / 3|SpinSmoke|
|1|SpinNoSmoke|
|2 / 4|KO|
|5|SpinKO|
|6|Shrink|
|7|ShrinkNoSmoke|
|8|None|
|9|Sink|
|10|ExplodeAnim|
|11|DropSprites|
|12|TODO: ???|

#### Exp logic
The `exp` field has particularily complex logic to determine its value:

- It is always 0 if the sent noexp value is true
- It is left at the default value of 0 if instance.`partylevel` is at least 27 (it's maxed) or if [flag](../Flags%20arrays/flags.md) 613 is true (RUIGEE is active)
- If the `fixedexp` field is true, then the amount is always the raw one coming from `enemydata` with no modifications to it
- If none of the cases above applied, the result is the return of GetEXP with the `exp` field from `enemydata`, the instance.`partylevel` and the enemy id (the base id if it's a variant)

The return of GetEXP implies even more logic where the base `exp` field can be changed. Here is the process used to calculate the final value:

1. The base EXP amount is determined. It's the same than the sent exp value unless the enemy is a `WaspTrooper` or `WaspHealer` which can increase this value:
    - If the current [map](../Enums%20and%20IDs/Maps.md) is `MetalLake` or the [area](../Enums%20and%20IDs/librarystuff/Areas.md) is `Wasp Kingdom Hive`, the value is multiplied by 1.65 and then ceiled
    - Otherwise, if the current [area](../Enums%20and%20IDs/librarystuff/Areas.md) is `Rubber Prison`, the value is multiplied by 2.5 ceiled. On top of this, if it's a `WaspHealer`, 10 is added to the amount
2. The value gets multiplied by map.`expmulti` which is a field that must be defined on the map's prefab. The result isn't rounded yet
3. The value gets subtracted by (level - 1) * 2.5 and the result is clamped from 1.0 to 99.0 then floored
5. The now integer value is decremented and returned

Additionally, GetEnemyData can also change yet again the value after it was calculated in 2 special cases (they both can happen and stack and they can happen regardless of how the initial `exp` value was determined):

1. [flag](../Flags%20arrays/flags.md) 162 is true (during a B.O.S.S or Cave Of Trials session): the value gets multiplied by 0.2, floored and then clamped from 0 to 5
2. createentity is true, [flag](../Flags%20arrays/flags.md) 166 is false (EX mode isn't active on the B.O.S.S. system) and the `battleentity` is a `hologram` (see the section above about the `battleentity` initialisation for when this happens): the value is multiplied by 0.1 and floored

Finally, [StartBattle](../Battle%20system/StartBattle.md) can further change the `exp` If all of the following conditions are true:

- The [enemy](../Enums%20and%20IDs/Enemies.md) is a `Krawler`, `CursedSkull` or `Cape`
- battleentity.`forcefire` is true or we are in the `GiantLair` [area](../Enums%20and%20IDs/librarystuff/Areas.md) except for the `GiantLairFridgeInside` [map](../Enums%20and%20IDs/Maps.md)
- instance.`partylevel` is less than 27 (meaning it's not maxed)
- [flags](../Flags%20arrays/flags.md) 613 is false (RUIGEE is inactive)

If that happens, the `exp` is incremented by the floored result of a lerp from 10.0 to 3.0 with a factor of instance.`partylevel` / 27.0

#### Special fields logic
Additionally, some `BattleData` fields have special logic attached to them:

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

- The `Hard Mode` [Medal](../Enums%20and%20IDs/Medal.md) is equipped
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

### About the portrait srpite index
This field is overridden to 225 for a `Cape`, 226 for a `Krawler` and 227 for a `CursedSkull` if [flag](../Flags%20arrays/flags.md) 664 is true (Approached the oven during Chapter 7). This overrides the portrait sprite to include the fire variants of the [enemy](../Enums%20and%20IDs/Enemies.md)

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
