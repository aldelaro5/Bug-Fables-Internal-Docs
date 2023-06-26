# Enemies

[Enemies](../Enums%20and%20IDs/Enemies.md) data are split in 3 TextAssets in the game loaded on boot:

* By LoadEssentials:
  * `Ressources/data/EnemyData`
* By SetVariables:
  * `Ressources/data/TattleList`
  * `EnemyTattle` from the corresponding dialogue directory of the current [languageid](../SetText/languageid.md)

`EnemyData` is also loaded on the Start and Update of the PauseMenu.

## `EnemyData` data

The asset contains one line per [Medal](../Enums%20and%20IDs/Medal.md) whose id corresponds to the line index. Each line contains fields separated by `@`:

|Loaded index|Name|Type|Description|
|------------|----|----|-----------|
|0|battleentity.animid|int|The [AnimIDs](../Enums%20and%20IDs/AnimIDs.md) of the entity bound to this enemy|
|1|hp|int|The base max HP of the enemy|
|2|def|int|The base defense of the enemy|
|3|exp|int|The base amount of exploration points this enemy gives|
|4|money|int|The base amount of berries this enemy drops|
|5|cursoroffset.x|float|The x component of the offset to apply to the selection cursor upon targeting the enemy|
|6|cursoroffset.y|float|The y component of the offset to apply to the selection cursor upon targeting the enemy|
|7|cursoroffset.z|float|The z component of the offset to apply to the selection cursor upon targeting the enemy|
|8|poisonres|int|The poison resistance of the enemy|
|9|freezeres|int|The freeze resistance of the enemy|
|10|numbres|int|The numb resistance of the enemy|
|11|sleepres|int|The sleep resistance of the enemy|
|12|size|float|The size scaling of the enemy|
|13|battleentity.freezesize.x|float|The x component of the ice cube size when the entity is frozen|
|14|battleentity.freezesize.y|float|The y component of the ice cube size when the entity is frozen|
|15|battleentity.freezesize.z|float|The z component of the ice cube size when the entity is frozen|
|16|battleentity.freezeoffset.x|float|The x component of the ice cube offset from the entity's center when the entity is frozen|
|17|battleentity.freezeoffset.y|float|The y component of the ice cube offset from the entity's center when the entity is frozen|
|18|battleentity.freezeoffset.z|float|The z component of the ice cube offset from the entity's center when the entity is frozen|
|19|position|BattlePosition (string or int)|The battle position of the enemy on the battle field|
|20|battleentity.height|float|The height of the entity|
|21|battleentity.bobspeed|float|The speed at which the entity bobs|
|22|battleentity.bobrange|float|By how much the entity bobs|
|23|weakness|`{` separated list of AttackProperty (int or string)|The list of attack properties the enemy is weak to|
|24|weight|float|???|
|25|Base [Enemies](../Enums%20and%20IDs/Enemies.md) id (loaded as animid)|int|The [Enemies](../Enums%20and%20IDs/Enemies.md) id to load most of the fields from (this is used for an enemy variation, see below for details), can be omitted by putting any negative number|
|26|eventondeath|int|The EventDialogue to trigger when the enemy dies?|
|27|moves|int|The amount of actions the enemy has normally each turn during the enemy phase|
|28|notaunt|bool|Tells if the enemy can be taunted?|
|29|cantfall|bool|Tells if the enemy cannot fall?|
|30|fixedexp|bool|Tells if the exp field should not be scaled and is a fixed number|
|31|notired|bool|???|
|32|hidehp|bool|Tells if the HP of the enemy should always be hidden even after spying|
|33|deathtype|int|The manner in which the enemy dies?|
|34|chargeonotherenemy|`;` separated list of int|The ? in which the enemy will get a charge when attacked|
|35|hardatk|int|The amount to increase the attack on Hard Mode or EX|
|36|Hard max HP increase|int|The base amount to increase the max HP on Hard Mode, EX or HARDEST (this is NOT `hardhp` which stays at 0)|
|37|Hard defense increase|int|The base amount to increase the defense on Hard Mode, EX or HARDEST (this is NOT `hardef` which stays at 0)|
|38|defenseonhit|int|The amount to increase the defense on hit?|
|39|itemoffset.x|float|The x component of the offset to render an item relative to the enemy|
|40|itemoffset.y|float|The y component of the offset to render an item relative to the enemy|
|41|itemoffset.z|float|The z component of the offset to render an item relative to the enemy|
|42|?|bool|Changes the battleentity.basestate to 13 ???|
|43|Portrait sprite index|int|Index of the sprite in `Sprites/Items/EnemyPortraits` (if this is negative, this defaults to the enemy id)|
|44|notattle|bool|Tells if the enemy is not spy-able (this is overridden to true in Cave Of Trials if the enemy wasn't seen)|
|45|eventonfall|int|The EventDialogue to trigger when the enemy falls ???|
|46|onhitaction|int|??? (why is this field an int?)|
|47|actimmobile|bool|???|
|48|sizeonfreeze|float|The size of the enemy on freeze ??? (why is this separate than the other ones?)|

The data will be loaded by `GetEnemyData` into `enemydata[id, x]`, where `id` is the [Medal](../Enums%20and%20IDs/Medal.md) id and `x` is the loaded index. On the pause menu, every lines is loaded into `enemydata` all at once, but the meanings do not change.

Here are the valid BattlePosition:

|Value|Name|
|-----|----|
|0|Ground|
|1|Flying|
|2|OutOfReach|
|3|Random|
|4|Underground|

Here are the valid AttackProperty:

|Value|Name|
|-----|----|
|0|Pierce|
|1|Flip|
|2|Freeze|
|3|Poison|
|4|Numb|
|5|Sleep|
|6|AntiPierce|
|7|ToppleFirst|
|8|Magic|
|9|ToppleAirOnly|
|10|Atleast1|
|11|Atleast1pierce|
|12|LimitX10|
|13|NoExceptions|
|14|None|
|15|HornExtraDamage|
|16|MinusMagic|
|17|DieInOneHit|
|18|SurviveWith10|
|19|FlyOnFlip|
|20|Fire|
|21|MultiHitDefense|
|22|DefDownOnBlock|
|23|DefDownOnFlyHard|
|24|Ink|
|25|AtkDownOnBlock|
|26|Sticky|
|27|InkOnBlock|
|28|AlwaysSurvive|
|29|Numb1Turn|

All `battleentity` fields are loaded only when the `createentity` parameter is true. Additionally, some fields have special loading logic:

* `exp`: The field is left at 0 if the party's level is 27 or if RUIGEE is active ([flags](../Flags%20arrays/flags.md) 613). It will be loaded, but overridden to 0 if the `noexp` parameter is true. Additionally, the actual amount loaded is scaled using the current level, but there are special cases with WaspTrooper and WaspHealer depending on the current map and area which can increase by 1.65x or 2.5x (floored) the amount of exp before scaling it. Additionally, the final amount after scaling if applicable is decreased by 80% floored and then clamped from 0 to 5 if [flags](../Flags%20arrays/flags.md) 162 is true (during a B.O.S.S or Cave Of Trials session). This number than gets decreased by 90% floored if it is TANGYBUG or an unseen enemy during Cave Of Trials.
* `sizeonfreeze` If the loaded vector is not long enough (0.1 magnitude), it will default to be `size` * 0.25.
* `hardatk`, `harddef`, `hardhp`, `hp`, `def`: see the section below detailing how enemy stats difficulty scaling works.
* `position`: If its value is Random, then it will be determined randomly and uniformly between Ground and Flying, but there's an exception if the enemy is a Mushroom and [flags](../Flags%20arrays/flags.md) 24 is true (received the Turn Relay tutorial) in which case, the position field is Flying.
* Portrait sprite index: This field is overridden to 225 for a Cape, 226 for a Krawler and 227 for a CursedSkull if [flags](../Flags%20arrays/flags.md) 664 is true (Approached the oven during Chapter 7).

### Stats difficulty scaling

There are some fields in enemy data that are influenced by the different difficulty tiers the game has. This only happens If the Hard Mode [Medal](../Enums%20and%20IDs/Medal.md) is on (or HARDEST is active) or if EX mode is active, otherwise, `hardatk` is left at 0 and nothing changes from the basic loading. The effects are applied in order they appear in this section if they are applicable. 

It should be noted that `harddef` and `hardhp` are NEVER loaded from enemy data no matter what: their values are always 0 and they can be considered unused despite the game reading them for calculating true hp and defense in some places. This turns out equivalent because the base hp and defense are increased on load by the same amount anyway.

If the above conditions are met, this happens regardless:

* The hp is incremented by Hard Mode/EX/HARDEST max HP increase
* The defense is incremented by Hard Mode/EX/HARDEST defense increase
* `hardatk` is loaded with its respective value

On top of this, if HARDEST is active:

* `hardatk` is incremented once
* `hp` is increased by 15% ceiled
* `def` is incremented if it wasn't negative and [flags](../Flags%20arrays/flags.md) 300 is true (start of Chapter 4)

On top of all this, if EX mode is active ([flags](../Flags%20arrays/flags.md) 166), then there are additional effects if the enemy isn't WaspGeneral, KeyR, KeyL or Tablet:

* `hp` is increased by 15% ceiled. If this ends up above 90, it is then decreased by 15% ceiled.
* `hardatk` is increased by 15% ceiled
* `def` is clamped from 1 to 99

### Enemy variant

The base [Enemies](../Enums%20and%20IDs/Enemies.md) id field uses a feature to redirect most of the fields loaded to be the ones belonging to another enemy entirely. It is used to add an enemy variant which only shares some fields with its base enemy, but it has its own id. Here are the list of fields that are shared:

* `eventondeath`
* `battleentity.height`
* `battleentity.bobspeed`
* `battleentity.bobrange`
* `moves`
* `notaunt`
* `cantfall`
* `notired`
* `fixedexp`
* `position`
  There are however some fields that are excluded from this if the original [Enemies](../Enums%20and%20IDs/Enemies.md) id is FireKrawler, FireWarden, FireCape, IceKrawler or IceWarden and they will have the value of the base enemy instead of the actual one being loaded:
* `moves`
* `notaunt`
* `cantfall`
* `notired`
* `fixedexp`
* `position`

Every other loading logic or fields will be loaded as if it was the enemy of the corresponding base [Enemies](../Enums%20and%20IDs/Enemies.md) id. This imply that every other field of the actual enemy will be ignored and they can be left as dummy values.

## `EnemyTattle` data

The asset contains one line per [Enemies](../Enums%20and%20IDs/Enemies.md) whose id corresponds to the line index. Each line contains fields separated by `@`:

|Loaded index|Name|Type|Description|
|------------|----|----|-----------|
|0|Name|[SetText](../SetText/SetText.md) string|The name of the enemy|
|1|Biography|[SetText](../SetText/SetText.md) string|The biography of the enemy|
|2|Vi's spy|[SetText](../SetText/SetText.md) string|Vi's Spy dialogue|
|3|Kabbu's spy|[SetText](../SetText/SetText.md) string|Kabbu's Spy dialogue|
|4|Leif's spy|[SetText](../SetText/SetText.md) string|Leif's Spy dialogue|

The data will be loaded into `librarydata[1, id, x]` where `id` is the [Enemies](../Enums%20and%20IDs/Enemies.md) id and `x` is the loaded index. The name is also loaded into `enemynames`.

Enemies not meant to have a [Bestiary entry](../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md) by convention have only the name field defined and the rest are dummy fields. The biography's dummy value is typically "biotattle" as well as the spy text fields being "beetattle", "beetletattle" and "mothtattle" respectively.

During `GetEnemyData`, the typical value is the one loaded into `enemynames` but it will be menutext 59 (?????) during Cave Of Trials for an unseen enemy or if the enemy is FireKrawler, FireCape or FireWarden while [flags](../Flags%20arrays/flags.md) 664 is false (not yet approached the oven during Chapter 7).

## `TattleList` data

The asset contains the order to render the [Bestiary entry](../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md) in the [Library List Type](../ItemList/List%20Types%20Group%20Details/Library%20List%20Type.md) where each line contains one field:

|Name|Type|Description|
|----|----|-----------|
|Enemy id|int|The [Enemies](../Enums%20and%20IDs/Enemies.md) id of the enemy to render at the position of the line index|

The data will be loaded into `libraryorder[1, i]` where i is the line index.
