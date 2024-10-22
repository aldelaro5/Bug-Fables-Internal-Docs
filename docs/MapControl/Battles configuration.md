# Battles configuration in map
Battles can be configured from the MapControl which dictates how they will be done.

Here are the configuration fields involved:

|Name|Type|Init|Description|Default|
|---:|----|----|----------|-------|
|battlemap|MainManager.BattleMaps|From prefab|The default BattleMaps to use when BattleControl.[StartBattle](../Battle%20system/StartBattle.md) is called on this map with a stageid of -1. This is also the default BattleMaps used for a CardBattle started with a mapid of -1|`Grasslands1` (should be assigned to the logically accurate value)|
|battleleaftype|BattleLeafType|From prefab|The type of transition to use on this map when [StartBattle](../Battle%20system/StartBattle.md) is called which differs visually. If the value is `Bee`, the `BattleStart3` sound will play instead of `BattleStart0`|`Common`|
|expmulti|float|From prefab|An EXP amount multiplier that affects the EXP scaling of MainManager.[GetEXP](../TextAsset%20Data/Enemies%20data.md#exp-logic) while called on this map. This should NEVER be negative or unexpected behaviors can occur|1.0|
|battleleafcolor|Color|From prefab|The default color of the visual transition of `battleleaftype` when [StartBattle](../Battle%20system/StartBattle.md) is called and adv isn't 3 (not an enemy party advantage)|Pure green|
|nobattlemusic|bool|From prefab|If this is true, it will prevent [StartBattle](../Battle%20system/StartBattle.md) from changing the music. This is a way for a map to keep the music the game was already playing during the battle without interruption. NOTE: This field is involved in the [music playback issue](../General%20systems/Music%20playback.md#issue-with-musicresume)|false|

## [StartBattle](../Battle%20system/StartBattle.md) influence
The map has the following influences on the battle's start:

- If stage is -1, the `BattleMap` will be the `battlemap` field
- The battle transition can be specified with the `battleleaftype` field (the color can be specified with `battleleafcolor` if adv isn't 3 meaning it's not an enemy first strike)
- It's possible to configure battles to not change the music if `nobattlemusic` is true. NOTE: This field is involved in the [music playback issue](../General%20systems/Music%20playback.md#issue-with-musicresume)

## `expmulti`
This field only takes into effect when either awarding EXP or estimating it (for the purpose of the `BumpAttack` medal). It has an influence on the complex [exp logic](../TextAsset%20Data/Enemies%20data.md#exp-logic) by acting as a multiplier that is applied right before rank scaling applied. It's a way for the map to scale all EXP values. For this reasons, this MUST NOT be negative or unexpected behaviours will occur.

This field has no effects on MainManage.GetEXP for any of the following conditions:

- The noexp parameter is true
- `partylevel` is 27 or more (max rank)
- The enemy data's `fixedexp` field is true (it needs to not be scaled)
