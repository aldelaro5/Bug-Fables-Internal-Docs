# Skills data

Skills data are split in 2 TextAssets in the game loaded on boot by SetVariables: 

* `Ressources/data/SkillData`
* `Skills` from the corresponding dialogue directory of the current [languageid](../SetText/languageid.md)

## `SkillData` data

The asset contains one line per skill whose id corresponds to the line index. Each line contains fields separated by `@`:

|Loaded index|Name|Type|Description|
|------------|----|----|-----------|
|2|TP cost|int|The cost of the skill in TP|
|3|Attack area|AttackArea|Tells who this skills affects on usage|
|4|Usable by Vi|bool|Tells if the skill is usable by Vi|
|5|Usable by Kabbu|bool|Tells if the skill is usable by Kabbu|
|6|Usable by Leif|bool|Tells if the skill is usable by Leif|
|7|Target ground only|bool|Tells if the skill can only target grounded enemies|
|8|Target front only|bool|Tells if the skill can only target the front enemy|
|9|Action command|int|The id of the action command|
|10|Target alive only|bool|Tells if the skill can only target alive entities|
|11|Exclude self target|bool|Tells if the skill excludes targeting its user|
|12|Target fainted only|bool|Tells if the skill can only target fainted entities|

The data will be loaded into `skilldata[id, x]` where `id` is the skill id and `x` is the loaded index.

The attack area must be specified as a string or int representation of an AttackArea enum value. Here are the valid values, anything else is considered invalid:

|Value|Name|
|-----|----|
|0|SingleEnemy|
|1|AllEnemy|
|2|SingleAlly|
|3|AllParty|
|4|All|
|5|None|
|6|User|

## `Skills` data

The asset contains one line per skill whose id corresponds to the line index. Each line contains 2 fields separated by `@`:

|Loaded index|Name|Type|Description|
|------------|----|----|-----------|
|0|Name|[SetText](../SetText/SetText.md) string|The name of the skill|
|1|Description|[SetText](../SetText/SetText.md) string|The description of the skill|

The data will be loaded into `skilldata[id, x]` where `id` is the skill id and `x` is the loaded index.
