# BoardQuests

[BoardQuests](../Enums%20and%20IDs/BoardQuests.md) are split in 3 TextAssets in the game loaded on boot: 

* By LoadEssentials:
    * `Ressources/data/QuestChecks`
* By SetVariables:
    * `Ressources/data/BoardData`
    * `BoardQuests` from the corresponding dialogue directory of the current [languageid](../SetText/languageid.md)

## `QuestChecks` data

The asset contains one line per [BoardQuests](../Enums%20and%20IDs/BoardQuests.md) whose id corresponds to the line index. Each line contains one a `@` separated list of one field per element:

|Name|Type|Description|
|----|----|-----------|
|Prerequisite|int|\> 0: a required [flags](../Flags%20arrays/flags.md) slot for the quest to be true|
|---|---|= 0: No prerequisite if it's the first element, having seen [Areas](../Enums%20and%20IDs/librarystuff/Areas.md) 0 (Bugaria Outskirt) otherwise|
|---|---|\< 0: A required [Areas](../Enums%20and%20IDs/librarystuff/Areas.md) id to have been seen|

The data will be loaded into `questchecks[id, x]` where `id` is the [BoardQuests](../Enums%20and%20IDs/BoardQuests.md) id and `x` is the index of each elements of the list starting from 0.

It should be noted that a prerequisite of 0 is only supported as it being the only element to indicate there are no prerequisite. Everything after will be ignored as the first element being 0 takes priority.

## `BoardData` data

The asset contains one line per [BoardQuests](../Enums%20and%20IDs/BoardQuests.md) whose id corresponds to the line index. Each line contains fields separated by `@`:

|Loaded index|Name|Type|Description|
|------------:|----|----|-----------|
|3|Taken flags|int|The [flags](../Flags%20arrays/flags.md) slot telling if the quest was taken|
|4|Icon|int|Index of the sprite in `Sprites/Items/EnemyPortraits` to render using the [Quest Board List Type](../ItemList/List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md)|
|5|Difficulty|int|The amount of stars to render as a difficulty indicator using the [Quest Board List Type](../ItemList/List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md)|

The data will be loaded into `boardquestdata[id, x]` where `id` is the [BoardQuests](../Enums%20and%20IDs/BoardQuests.md) id and `x` is the loaded index.

## `BoardQuests` data

The asset contains one line per [BoardQuests](../Enums%20and%20IDs/BoardQuests.md) whose id corresponds to the line index. Each line contains fields separated by `@`:

|Loaded index|Name|Type|Description|
|------------:|----|----|-----------|
|0|Name|[SetText](../SetText/SetText.md) string|The name of the quest|
|1|Description|[SetText](../SetText/SetText.md) string|The description of the quest|
|2|Sender|[SetText](../SetText/SetText.md) string|The name of the sender of the quest|

The data will be loaded into `boardquestdata[id, x]` where `id` is the [BoardQuests](../Enums%20and%20IDs/BoardQuests.md) id and `x` is the loaded index
