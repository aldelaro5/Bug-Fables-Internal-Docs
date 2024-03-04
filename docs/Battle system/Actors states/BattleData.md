# BattleData
This is defined in MainManager, but it is primarily used for the battle system.

## Player party members fields
These fields only applies to player party members.

|Name|Type|Description|
|---|---|---|
|trueid|int|The [animid](../../Enums%20and%20IDs/AnimIDs.md#animids) of the player party member|
|basehp|int|The max HP after applying all the `statbonus` from the initial one (this is initially 7 except for the `Beetle` [animid](../../Enums%20and%20IDs/AnimIDs.md#animids) where it's 9)|
|hpt|int|The displayed HUD value of the `hp`|
|baseatk|int|The attack after applying all the `statbonus` from the initial one (initially 2)|
|basedef|int|The defense after applying all the `statbonus` from the initial one (initially 0)|
|skills|List<int>|The list of available [skills](../../Enums%20and%20IDs/Skills.md), set to the return of [RefreshSkills](../RefreshSkills.md) for this player party member|
|lockskills|bool|If true, prevents [skills](../../Enums%20and%20IDs/Skills.md) to be used|
|lockitems|bool|If true, prevents items to be used|
|locktri|bool|If true, prevents relay to be used|
|haspassed|bool|If true, this player party member has already relayed which prevents it to relay again|
|lockrelayreceive|bool|If true, prevents to be relayed to from another player party member|
|plating|bool|Tells if the `Plating` [medal](../../Enums%20and%20IDs/Medal.md) is equipped on this player party member|
|didnothing|bool|Tells if the player party member already has chosen to do nothing on a previous action in the same turn which prevents to benefits from some [medals](../../Enums%20and%20IDs/Medal.md)'s effects. See [Do Nothing](../Player%20UI/SetItem.md#2-do-nothing) for details|
|lockcantmove|bool|If true, prevents `cantmove` to be set to 0 (one action available) whenever the `MiracleMatter` [medal](../../Enums%20and%20IDs/Medal.md) triggers. It is set to false after the medal triggers|
|eatenby|EntityControl|If not null, this player party member is trapped via eating by the entity corresponding to the value (this is only used when a `Pitcher` [enemy](../../Enums%20and%20IDs/Enemies.md) eats a player party member and it not being null is treated the same as having an `hp` of 0 or below which is death)|

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
|fled|bool|If true, it indicates the enemy party member fled the battle which is seen the same way than if the `hp` reached 0 during [CheckDead](../Battle%20flow/Action%20coroutines/CheckDead.md) which means it's the same than dying, but with less logic involved to kill the enemy party member|
|destroyentity|bool|If the enemy party member `fled` and this is true while the fleeing is found by [CheckDead](../Battle%20flow/Action%20coroutines/CheckDead.md), the battleentity will be disabled as part of the killing process|
|alreadycounted|bool|Indicates to [CheckDead](../Battle%20flow/Action%20coroutines/CheckDead.md) that the enemy party member was already killed prior and therefore, shouldn't cause another increment of the [bestiary entry](../../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md)'s defeat counter the next time the enemy party member dies|
|eventondeath|int|The [EventDialogue](../Battle%20flow/EventDialogue.md) id to trigger during [CheckDead](../Battle%20flow/Action%20coroutines/CheckDead.md) when it detects that the enemy party member died|
|diebyitself|bool|If true, it means the enemy party member will die if all of the ones left have this field set to true automatically when [CheckDead](../Battle%20flow/Action%20coroutines/CheckDead.md) detects this situation after it had killed any other enemy party members. This is done by setting the `hp` to 0 manually before running a second check on all enemy party members which will kill them|
|deathtype|int|For an enemy, dictates the manner in which the enemy dies (this isn't a DeathType, see the [battleentity initialisation documentation](../../TextAsset%20Data/Enemies%20data.md#battleentity-special-initialisation) for more details)|
|eventonfall|int|The [EventDialogue](../Battle%20flow/EventDialogue.md) to trigger when the enemy party member falls to the ground when sustaining damage (meaning [Drop](../../Entities/EntityControl/Notable%20methods/Drop.md#drop) is called on the battleentity as a result of sustaining damage)|
|sizeonfreeze|float|The size returned by GetEnemySize when the enemy party member has the `Frozen` [condition](Conditions.md) instead of its `size`|
|initialsize|float|The initial `size` value of the enemy party member. `size` will be set to this value when [BreakIce](../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#breakice) is called on the battleentity|
|battlepos|Vector3|The resting battle position of the enemy party members used only in an [EventDialogue](../Battle%20flow/EventDialogue.md)|
|delayedcondition|List<int>|A list of [conditions](Conditions.md) (only the `BattleCondition` part) added by the damage pipeline that will be inflicted later un [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md) TODO: learn more during damage pipeline docs|
|data|int\[\]|General purpose data array for use in [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md)|
|ate|EntityControl|The entity this enemy party member is trapping via eating (this is only used for the `Pitcher` [enemy](../../Enums%20and%20IDs/Enemies.md))|
|position|BattlePosition|The logical battle position frequently used to determine target availability|
|extrastuff|Transform\[\]|A general purspose transforms array for use in [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md)|
|frostbitep|ParticleSystem|An instance of `Prefabs/Particles/Snowflakes` childed to the battleentity that is initialised whenever the `Freeze` [condition](Conditions.md) is added as a delayed condition|
|cantfall|bool|If true, the enemy party member cannot fall to the ground as a result of sustaining damage|
|notaunt|bool|If true, the enemy party member cannot be inflicted by the `Taunted` [condition](Conditions.md) from the player party|
|notired|bool|If true, the enemy will not cumulated any `tired` (meaning it can't gain exhaustion) whenever the `DoublePain` [medal](../../Enums%20and%20IDs/Medal.md) is not equipped|
|actimmobile|bool|??? TODO: involved in checks to see if the enemy is active|
|weakness|List<AttackProperty>|The list of properties that applies for attacks on this ??? TODO: it's not just weaknesses, for some reasons, alwayssurvive placed in this case make the enemy always survive, heavily involved in the damage pipeline|
|lockposition|bool|??? TODO: involved in the damage pipeline|

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
|cantmove|int|The amount of actor turns that needs to pass until an action is possible. 0 Means one action is possible and anything negative further adds possible actions on the same turn|
|tired|int|The amount of exhaustion cumulated|
|holditem|int|The [item](../../Enums%20and%20IDs/Items.md) id the actor is holding. Defaults to -1 which is no item held|
|cursoroffset|Vector3|The offset to apply to the cursor when selecting this|
|itemoffset|Vector3|The offset to render the item relative to the entity|
|condition|List<int\[\]>|The list of condition applied. Each element is an array of 2 elements, the first being the `BattleCondition` id and the second being the amount of actor turns it is in effect|
|battleentity|EntityControl|The entity used during the battle|
|isasleep|bool|Tells if the actor has a `Sleep` condition, set on HasCondition if that condition exist|
|isnumb|bool|Tells if the actor has a `Numb` condition, set on FixCondition which is called by SetCondition|
|entity|EntityControl|For a player party member, the overworld entity. For an enemy party member, the same than `battleentity`|
|charge|int|The amount of charges cumulated|
|turnssincedeath|int|The amount of actor turns since death, 0 if the actor is alive|
|turnsalive|int|The amount of actor turns since the battle started or their last death if any occured|
|frozenlastturn|bool|Tells if the actor still had the `Freeze` [condition](Conditions.md) after their last actor turn advance via [AdvanceTurnEntity](../Battle%20flow/AdvanceTurnEntity.md)|
|atkdownonloseatkup|bool|If true, the actor will get an `AttackDown` [condition](Conditions.md) inflicted the next time an `AttackUp` condition runs out of actor turns|
|moreturnnextturn|int|When above 0, on the next actor turn, the actor's `cantmove` will be decreased by it which gives it this amount of additional actions available|
|animid|int|For an enemy, the [enemy](../../Enums%20and%20IDs/Enemies.md) id. For a player, the same as the `trueid` (the actual [animid](../../Enums%20and%20IDs/AnimIDs.md) can be fetched via battleentity.`animid`)|
|helditem|SpriteRenderer|The `sprite` of the [item](../../Enums%20and%20IDs/Items.md) held by the actor whose id is `holditem`|

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
|noexpatstart|bool|UNUSED (intended for enemy party members only, only read during damage pipeline, but never written to so it always have the value false)|
