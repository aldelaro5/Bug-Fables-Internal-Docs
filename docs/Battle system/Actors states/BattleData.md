# BattleData
This is defined in MainManager, but it is primarily used for the battle system.

## Uncategorised fields
These fields's semantics haven't been found yet. They will be moved out of this section as they are figured out

|Name|Type|Description|
|---|---|---|
|eventondeath|int|For an enemy, the EventDialogue to trigger when it dies ???|
|deathtype|int|For an enemy, dictates the manner in which the enemy dies (this isn't a DeathType, see the [battleentity initialisation documentation](../../TextAsset%20Data/Enemies%20data.md#battleentity-special-initialisation) for more details) TODO: seems to have more to it ???|
|eventonfall|int|The EventDialogue to trigger when the enemy falls ???|
|turnssincedeath|int|???|
|turnsalive|int|???|
|moreturnnextturn|int|???|
|sizeonfreeze|float|For an enemy, the `size` when frozen ???|
|initialsize|float|???|
|battlepos|Vector3|The battle position ??? TODO: this seems to be the neutral position to return to after an action, recheck|
|delayedcondition|List<int>|???|
|data|int\[\]|General purpose data array ???|
|helditem|SpriteRenderer|???|
|ate|EntityControl|The entity this is currently eating ???|
|eatenby|EntityControl|The entity currently eating this one ???|
|position|BattlePosition|The current battle position ???|
|weakness|List<AttackProperty>|The list of properties that applies for attacks on this ??? TODO: it's not just weaknesses, for some reasons, alwayssurvive placed in this case make the enemy always survive...|
|extrastuff|Transform\[\]|???|
|frostbitep|ParticleSystem|???|
|cantfall|bool|Tells if the enemy cannot fall ???|
|notaunt|bool|Tells if the enemy can't be taunted ???|
|notired|bool|???|
|fled|bool|???|
|actimmobile|bool|??? TODO: involved in checks to see if the enemy is active|
|diebyitself|bool|???|
|noexpatstart|bool|???|
|atkdownonloseatkup|bool|???|
|lockposition|bool|???|
|alreadycounted|bool|???|
|destroyentity|bool|???|
|frozenlastturn|bool|???|
|lockcantmove|bool|???|
|animid|int|For an enemy, the [enemy](../../Enums%20and%20IDs/Enemies.md) id. For a player, ???|

## Player party members fields
These fields only applies to player party members.

|Name|Type|Description|
|---|---|---|
|trueid|int|The [animid](../../Enums%20and%20IDs/AnimIDs.md#animids) of the player party member|
|basehp|int|The max HP after applying all the `statbonus` from the initial one (this is normally 7 except for the `Beetle` [animid](../../Enums%20and%20IDs/AnimIDs.md#animids) where it's 9)|
|hpt|int|The displayed HUD value of the `hp`|
|baseatk|int|The attack after applying all the `statbonus` from the initial one (normally 2)|
|basedef|int|The defense after applying all the `statbonus` from the initial one (normally 0)|
|skills|List<int>|The list of available skills, set to the return of [RefreshSkills](../RefreshSkills.md) for this player party member|
|lockskills|bool|If true, prevents skills to be used|
|lockitems|bool|If true, prevents items to be used|
|locktri|bool|If true, prevents relay to be used|
|haspassed|bool|If true, this player party member has already relayed which prevents it to relay again|
|lockrelayreceive|bool|If true, prevents to be relayed to from another player party member|
|plating|bool|Tells if the `Plating` [medal](../../Enums%20and%20IDs/Medal.md) is equipped on this player party member|
|didnothing|bool|Tells if the player party member already has chosen to do nothing on a previous action in the same turn which prevents to benefits from some [medals](../../Enums%20and%20IDs/Medal.md)'s effects. See [Do Nothing](../Player%20UI/SetItem.md#2-do-nothing) for details|

## Enemy party members fields
These fields only applies to enemy party members.

|Name|Type|Description|
|---|---|---|
|exp|int|The amount of EXP given when defeated (see the [EXP documentation](../../TextAsset%20Data/Enemies%20data.md#exp-logic) for details)|
|money|int|The base amount of berries this enemy drops|
|moves|int|The amount of actions allowed per turn|
|hardatk|int|The attack gained when the [hard dfficulty scaling](../../TextAsset%20Data/Enemies%20data.md#stats-difficulty-scaling) applies (this is 0 if it doesn't)|
|onhitaction|int|If it's 1, `hitaction` will be set to true when hit. If it's 2, it will only if `position` is `Flying` and if it's 3, only if it's `Ground`|
|chargeonotherenemy|int\[\]|The list of [enemy](../../Enums%20and%20IDs/Enemies.md) ids that when hit, will cause this enemy to have its `hitaction` set to true|
|size|float|The width used for calculating positioning in battle|
|entityname|string|The [SetText](../../SetText/SetText.md) string of its name|
|fixedexp|bool|Tells if the exp field should not be scaled and is a fixed number (see the [EXP documentation](../../TextAsset%20Data/Enemies%20data.md#exp-logic) for details)|
|hidehp|bool|Tells if its battleentity.`hpbar` should always be hidden even after spying|
|isdefending|bool|Tells if the enemy is guarding which increases its effective defense by `defenseonhit`|
|notattle|bool|Tells if the enemy is not spy-able (this can be overriden to true, see the documentation about [special fields logic](../../TextAsset%20Data/Enemies%20data.md#special-fields-logic) for details)|
|hitaction|bool|When set to true, the enemy will perform a [hitation](../Battle%20flow/Update.md#enemies-hitaction) on the next controlled update flow|
|defenseonhit|int|The amount to increase the defense when `isdefending` is true|
|weight|float|A modifier to apply when animating the enemy when it gets hurt by certain attacks|

## Common actor fields
These fields applies to actors for either parties.

|Name|Type|Description|
|---|---|---|
|hp|int|The HP, (for a player party member, this is clamped from 0 to `maxhp`)|
|maxhp|int|The maximum amount of HP (for a player party member, this is `basehp` after all [medal](../../Enums%20and%20IDs/Medal.md) effects applied)|
|atk|int|The attack (for a player party member, this is `baseatk` after all [medal](../../Enums%20and%20IDs/Medal.md) effects applied)|
|def|int|The base defense (for a player party member, this is `basedef` after all [medal](../../Enums%20and%20IDs/Medal.md) effects applied)|
|freezeres|int|The freeze resistance|
|poisonres|int|The poison resistance|
|numbres|int|The numb resistance|
|sleepres|int|The sleep resistance|
|cantmove|int|The amount of turns that needs to pass until an action is possible for the next turn. 0 Means one action is possible and anything negative further adds possible actions on the same turn|
|tired|int|The amount of exhaustion cumulated|
|holditem|int|The [item](../../Enums%20and%20IDs/Items.md) id the actor is holding. Defaults to -1 which is no item held|
|cursoroffset|Vector3|The offset to apply to the cursor when selecting this|
|itemoffset|Vector3|The offset to render the item relative to the entity|
|condition|List<int\[\]>|The list of condition applied. Each element is an array of 2 elements, the first being the `BattleCondition` id and the second being the amount of turns it is in effect|
|battleentity|EntityControl|The entity used during the battle|
|isasleep|bool|Tells if the actor has a `Sleep` condition, set on HasCondition if that condition exist|
|isnumb|bool|Tells if the actor has a `Numb` condition, set on FixCondition which is called by SetCondition|
|entity|EntityControl|For a player party member, the overworld entity. For an enemy party member, the same than `battleentity`|
|charge|int|The amount of charges cumulated|

## Unused fields
These fields are never referenced or never used in any meaningful ways.

|Name|Type|Description|
|---|---|---|
|tiredpart|ParticleSystem|UNUSED|
|noblock|bool|UNUSED|
|pointer|int|UNUSED|
|turnsnodamage|int|UNUSED|
|hardhp|int|UNUSED|
|harddef|int|UNUSED|
|lv|int|UNUSED|
|id|int|UNUSED (only read from [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md), but never written to so it stays at 0)|
