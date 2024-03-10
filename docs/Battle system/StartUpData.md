# StartUpData
All these values are saved into `sdata` when [StartBattle](StartBattle.md) is called while `saveddata` was false.

|Name|Type|Description|
|----|----|-----------|
|flagvar|int\[\]|All the [flagvar](../Flags%20arrays/flagvar.md)|
|enemies|int\[\]|The sent enemyids value to StartBattle (after [EnemyCheck](StartBattle%20phases/Pre%20haltbattleload.md#enemycheck) ran when calledfrom doesn't exists or its `eventid` is 0 or beloe)|
|items|int\[\]|instance.`items[0]` which are all the standard [items](../Enums%20and%20IDs/Items.md#items) inventory|
|hp|int\[\]|All the `playerdata`'s `hp`|
|atk|int\[\]|All the `playerdata`'s `atk`|
|def|int\[\]|All the `playerdata`'s `def`|
|keyitem|int\[\]|instance.`items[1]` which are all the key [items](../Enums%20and%20IDs/Items.md#items) inventory|
|flags|bool\[\]|All the [flags](../Flags%20arrays/flags.md)|
|partyitemuse|bool\[\]|All the `playerdata`'s `lockitems`|
|stage|int|The sent stageid from StartBattle|
|tp|int|instance.`tp`|
|adv|int|The sent adv from StartBattle|
|maxtp|int|instance.`maxtp`|
|music|string|This is either the sent music from StartBattle or MainManager.`music[0]`.clip.name. It will set it to the former if the sent music isn't null / empty or MainManager.`music[0]` is null or not playing. It will be set to the latter otherwise|
|called|NPCControl|The calledfrom sent from StartBattle|
|encounter|int\[,\]|instance.`enemyencounter`|
|psc|List<int\[\]\[\]>|All the `playerdata`'s [conditions](Actors%20states/Conditions.md#conditions)|
|esc|List<int\[\]\[\]>|All the `enemydata`'s [conditions](Actors%20states/Conditions.md)|
|enemyweakness|List<[AttackProperty](Damage%20pipeline/AttackProperty.md)\[\]>|All the `enemydata`'s [weakness](Actors%20states/Enemy%20features.md) (if the `weakness` field is null, null is the only element set)|
