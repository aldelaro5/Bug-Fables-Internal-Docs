# Pick
This enum describes the current top level interface the player is naviguating. The current one is stored in the `currentaction` field. It can be thought as the top level interface while [Actions](Actions.md) is the sub level one.

|Value|Name|Description|
|-----|----|-----------|
|0|BaseAction|The main vine action menu|
|1|SelectEnemy|Selecting an enemy party member|
|2|SelectPlayer|Selecting a player party member|
|3|SkillList|Selecting a [skill](../../Enums%20and%20IDs/Skills.md) in its [ItemList](../../ItemList/ItemList.md)|
|4|ItemList|Selecting an [item](../../Enums%20and%20IDs/Items.md) in its [ItemList](../../ItemList/ItemList.md)|
|5|StrategyList|Selecting a strategy in its [ItemList](../../ItemList/ItemList.md)|
|6|Relay|UNUSED|
|7|Chompy|The chompy vine menu used in [Chompy](../Battle%20flow/Action%20coroutines/Chompy.md)|
