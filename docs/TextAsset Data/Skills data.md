# Skills data

Skills data are split in 2 TextAssets in the game loaded on boot by SetVariables: 

* `Ressources/data/SkillData`
* `Skills` from the corresponding dialogue directory of the current [languageid](../SetText/languageid.md)

## `SkillData`

The asset contains one line per [skill](../Enums%20and%20IDs/Skills.md) whose id corresponds to the line index. Each line contains fields separated by `@`:

|Loaded index|Name|Type|Description|
|------------:|----|----|-----------|
|2|Cost|int|The cost of the skill in TP if not negative or in HP if it is negative|
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
|-----:|----|
|0|SingleEnemy|
|1|AllEnemy|
|2|SingleAlly|
|3|AllParty|
|4|All|
|5|None|
|6|User|

### About the usable by fields
These 3 fields does tell if the skill can be used and it will be enforced when trying to confirm its selection. However, they do not actually tell if the skill is available in battle because this is managed in a different way where this information is hardcoded in the return of [RefreshSkills](../Battle%20system/RefreshSkills.md). This means that there is 2 different sources of truth: RefreshSkills enforces that the skill is available for usage given a particular member and the data field enforces that it is usable once it is available by the same member. It implies that for the skills usability to correctly function, both RefreshSkills and the usable by fields should have matching information. Failure to do so will result in a skill being unavailable, but usable or vice versa which isn't logical.

## `Skills` data

The asset contains one line per skill whose id corresponds to the line index. Each line contains 2 fields separated by `@`:

|Loaded index|Name|Type|Description|
|------------:|----|----|-----------|
|0|Name|[SetText](../SetText/SetText.md) string|The name of the skill|
|1|Description|[SetText](../SetText/SetText.md) string|The description of the skill|

The data will be loaded into `skilldata[id, x]` where `id` is the skill id and `x` is the loaded index.
