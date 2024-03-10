# BattleData
This is defined in MainManager, but it is primarily used for the battle system. It contains all the data associated with an actor, but it primarily augments an [EntityControl](../../Entities/EntityControl/EntityControl.md) which is stored in the battleentity field (or the entity field for the overworld version for player party members).

Fields can be intended to be used for only player party members, only enemy party members or any actors depending on the specific field.

## Player party members fields
These fields only applies to player party members.

### Addressing

|Name|Type|Description|
|---:|---|---|
|trueid|int|The [animid](../../Enums%20and%20IDs/AnimIDs.md#animids) of the player party member (learn more why this is equivalent to the `animid` in the [playerdata addressing](../playerdata%20addressing.md) documentation)|

### Stats

|Name|Type|Description|
|---:|---|---|
|basehp|int|The max HP after applying all the `statbonus` from the initial one (this is initially 7 except for the `Beetle` [animid](../../Enums%20and%20IDs/AnimIDs.md#animids) where it's 9)|
|hpt|int|The displayed HUD value of the `hp`|
|baseatk|int|The attack after applying all the `statbonus` from the initial one (initially 2)|
|basedef|int|The defense after applying all the `statbonus` from the initial one (initially 0)|

### Abilities

|Name|Type|Description|
|---:|---|---|
|skills|List<int>|The list of available [skills](../../Enums%20and%20IDs/Skills.md), set to the return of [RefreshSkills](../RefreshSkills.md) for this player party member|
|lockskills|bool|If true, prevents [skills](../../Enums%20and%20IDs/Skills.md) to be used|
|lockitems|bool|If true, prevents [items](../../Enums%20and%20IDs/Items.md) to be used|
|locktri|bool|If true, prevents relay to be used|
|haspassed|bool|If true, this player party member has already relayed which prevents it to relay again|
|lockrelayreceive|bool|If true, prevents to be relayed to from another player party member|
|lockcantmove|bool|If true, prevents `cantmove` to be set to 0 (one action available) whenever the `MiracleMatter` [medal](../../Enums%20and%20IDs/Medal.md) triggers. It is set to false after the medal triggers|
|didnothing|bool|Tells if the player party member already has chosen to [do nothing](../Player%20UI/ItemList%20confirmation%20handling/Battle%20strategy%20list%20type.md#2-do-nothing) on a previous action in the same main turn which prevents to benefits from some [medals](../../Enums%20and%20IDs/Medal.md)'s effects|

### Special states

|Name|Type|Description|
|---:|---|---|
|plating|bool|Tells if the `Plating` [medal](../../Enums%20and%20IDs/Medal.md) is equipped on this player party member and it is active|
|eatenby|EntityControl|If not null, this player party member is trapped via [eating](BattleCondition/Eaten.md) by the entity corresponding to the value (this is only used when a `Pitcher` [enemy](../../Enums%20and%20IDs/Enemies.md) eats a player party member and it not being null is treated the same as having an `hp` of 0 or below which is death)|

## Enemy party members fields
These fields only applies to enemy party members. Some of them are relayed to specific [enemy features](Enemy%20features.md) which are documented separately.

### Core state

|Name|Type|Description|
|---:|---|---|
|weakness|List<[AttackProperty](../Damage%20pipeline/AttackProperty.md)>|The list of properties that applies on this enemy party member during the damage pipeline|
|position|[BattlePosition](BattlePosition.md)|The logical battle position frequently used to determine target availability|
|[delayedcondition](Delayed%20condition.md)|List<int>|A list of [conditions](Conditions.md) (only the `BattleCondition` part) added by the damage pipeline that will be inflicted later un [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md)|

### Stats

|Name|Type|Description|
|---:|---|---|
|hardatk|int|The attack gained when the [hard dfficulty scaling](../../TextAsset%20Data/Enemies%20data.md#stats-difficulty-scaling) applies (this is 0 if it doesn't)|

### Rewards

|Name|Type|Description|
|---:|---|---|
|exp|int|The amount of EXP given when defeated (see the [EXP documentation](../../TextAsset%20Data/Enemies%20data.md#exp-logic) for details)|
|fixedexp|bool|Tells if the exp field should not be scaled and is a fixed number (see the [EXP documentation](../../TextAsset%20Data/Enemies%20data.md#exp-logic) for details)|
|money|int|The base amount of berries this enemy party member drops|

### Action features

|Name|Type|Description|
|---:|---|---|
|[moves](Enemy%20features.md#moves)|int|The amount of actions allowed per turn|
|[actimmobile](Enemy%20features.md#actimmobile)|bool|When true, it allows the enemy party member to act even if they have a stopping [condition](Conditions.md). Check the feature documentation to learn more|
|[hitaction](Enemy%20features.md#hitaction)|bool|When set to true, the enemy party member will perform a hitation on the next controlled update flow|
|[onhitaction](Enemy%20features.md#onhitaction)|int|If it's 1, [hitaction](Enemy%20features.md#hitaction) will be set to true when hit. If it's 2, it will only if [position](BattlePosition.md#battleposition) is `Flying` and if it's 3, only if it's `Ground`|
|[chargeonotherenemy](Enemy%20features.md#chargeonotherenemy)|int\[\]|The list of [enemy](../../Enums%20and%20IDs/Enemies.md) ids that when hit, will cause this enemy to have its [hitaction](Enemy%20features.md#hitaction) set to true|

### Guard feature

|Name|Type|Description|
|---:|---|---|
|[defenseonhit](Enemy%20features.md#defenseonhit-and-isdefending)|int|The amount to increase the defense when [isdefending](Enemy%20features.md#isdefending) is true|
|[isdefending](Enemy%20features.md#isdefending)|bool|Tells if the enemy party member is guarding which increases its effective defense by [defenseonhit](Enemy%20features.md#defenseonhit-and-isdefending) (It can also affect [hitaction](Enemy%20features.md#hitaction) if it's -1, check the feature documentation to learn more)|

### Conditional [EventDialogue](../Battle%20flow/EventDialogue.md)

|Name|Type|Description|
|---:|---|---|
|[eventondeath](Enemy%20features.md#eventondeath)|int|The [EventDialogue](../Battle%20flow/EventDialogue.md) id to trigger during [CheckDead](../Battle%20flow/Action%20coroutines/CheckDead.md) when it detects that the enemy party member died. -1 means no EventDialogue will occur|
|[eventonfall](Enemy%20features.md#eventonfall)|int|The [EventDialogue](../Battle%20flow/EventDialogue.md) to trigger when the enemy party member falls to the ground when sustaining damage (meaning [Drop](../../Entities/EntityControl/Notable%20methods/Drop.md#drop) is called on the battleentity as a result of sustaining damage). -1 means no EventDialogue will occur|

### Override features

|Name|Type|Description|
|---:|---|---|
|[cantfall](Enemy%20features.md)|bool|If true, the enemy party member cannot fall to the ground as a result of sustaining damage in [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md)|
|lockposition|bool|Similar to [cantfall](Enemy%20features.md#cantfall) for [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md) specifically, but it takes effect even if the enemy party member's [position](BattlePosition.md) isn't `Flying`|
|[notaunt](Enemy%20features.md#notaunt)|bool|If true, the enemy party member cannot be inflicted by the [Taunted](BattleCondition/Taunted.md) condition from the player party (this doesn't remove the logic, see the documentation of the feature to learn more)|
|[notattle](Enemy%20features.md#notattle)|bool|Tells if the enemy party member is not spy-able (this can be overriden to true, see the documentation about [special fields logic](../../TextAsset%20Data/Enemies%20data.md#special-fields-logic) for details)|

### Death behavior

|Name|Type|Description|
|---:|---|---|
|[fled](Enemy%20features.md#fled)|bool|If true, it indicates the enemy party member fled the battle which is seen the same way than if the `hp` reached 0 during [CheckDead](../Battle%20flow/Action%20coroutines/CheckDead.md) which means it's the same than dying, but with less logic involved to kill the enemy party member|
|notired|bool|If true, the enemy party member will not cumulated any `tired` (meaning it can't gain exhaustion) whenever the `DoublePain` [medal](../../Enums%20and%20IDs/Medal.md) is not equipped. NOTE: This doesn't cover HARDEST correctly, see the [EndEnemyTurn](../Battle%20flow/EndEnemyTurn.md) documentation to learn more|
|[deathtype](Enemy%20features.md#deathtype)|int|Dictates the manner in which the enemy party member dies|
|[diebyitself](Enemy%20features.md#diebyitself)|bool|If true, it means the enemy party member will die if all of the ones left have this field set to true automatically when [CheckDead](../Battle%20flow/Action%20coroutines/CheckDead.md) detects this situation after it had killed any other enemy party members. This is done by setting the `hp` to 0 manually before running a second check on all enemy party members which will kill them|
|alreadycounted|bool|Indicates to [CheckDead](../Battle%20flow/Action%20coroutines/CheckDead.md) that the enemy party member was already killed prior and therefore, shouldn't cause another increment of the [bestiary entry](../../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md)'s defeat counter the next time the enemy party member dies|
|destroyentity|bool|If the enemy party member `fled` and this is true while the fleeing is found by [CheckDead](../Battle%20flow/Action%20coroutines/CheckDead.md), the battleentity will be disabled as part of the killing process|

### Size calculation

|Name|Type|Description|
|---:|---|---|
|size|float|The width used for calculating positioning in battle|
|initialsize|float|The initial `size` value of the enemy party member. `size` will be set to this value when [BreakIce](../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#breakice) is called on the battleentity|
|[sizeonfreeze](Enemy%20features.md#sizeonfreeze)|float|The size returned by GetEnemySize when the enemy party member has the [Freeze](BattleCondition/Freeze.md#freeze) condition instead of its `size`|

### Visual modifiers

|Name|Type|Description|
|---:|---|---|
|[hidehp](Enemy%20features.md#hidehp)|bool|Tells if its battleentity.`hpbar` should always be hidden even after spying|
|[weight](Enemy%20features.md#weight)|float|A visual modifier to apply when animating the enemy when it gets hurt by certain attacks|

### General purpose

|Name|Type|Description|
|---:|---|---|
|[data](Enemy%20features.md#data)|int\[\]|General purpose data array for use in [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md)|
|[extrastuff](Enemy%20features.md)|Transform\[\]|A general purspose transforms array for use in [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md)|

### Visual rendering

|Name|Type|Description|
|---:|---|---|
|battlepos|Vector3|The resting battle position of the enemy party members used only in an [EventDialogue](../Battle%20flow/EventDialogue.md)|
|entityname|string|The [SetText](../../SetText/SetText.md) string of its name|

### [Conditions](Conditions.md) specific

|Name|Type|Description|
|---:|---|---|
|ate|EntityControl|The entity this enemy party member is trapping via [eating](BattleCondition/Eaten.md) (this is only used for the `Pitcher` [enemy](../../Enums%20and%20IDs/Enemies.md))|
|frostbitep|ParticleSystem|An instance of `Prefabs/Particles/Snowflakes` childed to the battleentity that is initialised whenever the [Freeze](BattleCondition/Freeze.md) condition is added as a [delayed condition](Delayed%20condition.md#delayed-condition)|

## Common actor fields
These fields applies to actors from either parties.

### [EntityControl](../../Entities/EntityControl/EntityControl.md)

|Name|Type|Description|
|---:|---|---|
|battleentity|EntityControl|The entity used during the battle|
|entity|EntityControl|For a player party member, the overworld entity. For an enemy party member, the same than `battleentity`|

### Identification

|Name|Type|Description|
|---:|---|---|
|animid|int|For an enemy, the [enemy](../../Enums%20and%20IDs/Enemies.md) id. For a player, the same as the `trueid` (learn more in the [playerdata addressing](../playerdata%20addressing.md) documentation)|

### Stats

|Name|Type|Description|
|---:|---|---|
|hp|int|The HP, (for a player party member, this is clamped from 0 to `maxhp`)|
|maxhp|int|The maximum amount of HP (for a player party member, this is `basehp` after all [medal](../../Enums%20and%20IDs/Medal.md) effects applied)|
|atk|int|The attack (for a player party member, this is `baseatk` after all [medal](../../Enums%20and%20IDs/Medal.md) effects applied)|
|def|int|The base defense (for a player party member, this is `basedef` after all [medal](../../Enums%20and%20IDs/Medal.md) effects applied)|

### Actor turn tracking

|Name|Type|Description|
|---:|---|---|
|cantmove|int|The amount of actor turns that needs to pass until an action is possible. 0 Means one action is possible and anything negative further adds possible actions on the same main turn|
|moreturnnextturn|int|When above 0, on the next main turn, the actor's `cantmove` will be decreased by it which gives it this amount of additional actor turns available|

### [Conditions] tracking

|Name|Type|Description|
|---:|---|---|
|[condition](Conditions.md#conditions)|List<int\[\]>|The list of condition applied. Each element is an array of 2 elements, the first being the `BattleCondition` id and the second being the amount of actor turns it is in effect|
|freezeres|int|The [Freeze](BattleCondition/Freeze.md) resistance|
|poisonres|int|The [Poison](BattleCondition/Poison.md) resistance|
|numbres|int|The [Numb](BattleCondition/Numb.md) resistance|
|sleepres|int|The [Sleep](BattleCondition/Sleep.md) resistance|
|isasleep|bool|Tells if the actor has a [Sleep](BattleCondition/Sleep.md) condition, updated at multiple places|
|isnumb|bool|Tells if the actor has a [Numb](BattleCondition/Numb.md) condition, updated at multiple places|
|frozenlastturn|bool|Tells if the actor still had the [Freeze](BattleCondition/Freeze.md) condition after their last actor turn advance via [AdvanceTurnEntity](../Battle%20flow/AdvanceTurnEntity.md)|
|atkdownonloseatkup|bool|If true, the actor will get an [AttackDown](BattleCondition/AttackDown.md) condition inflicted the next time an [AttackUp](BattleCondition/AttackUp.md) condition runs out of main turns|

### Damage modifiers

|Name|Type|Description|
|---:|---|---|
|tired|int|The amount of exhaustion cumulated which have an impact in [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md)|
|charge|int|The amount of charges cumulated which have an impact in [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md)|

### [Item](../../Enums%20and%20IDs/Items.md) holding

|Name|Type|Description|
|---:|---|---|
|holditem|int|The [item](../../Enums%20and%20IDs/Items.md) id the actor is holding. Defaults to -1 which is no item held. For enemy party members, see the [holditem](Enemy%20features.md#holditem-and-helditem) feature documentation|
|helditem|SpriteRenderer|The `sprite` of the [item](../../Enums%20and%20IDs/Items.md) held by the actor whose id is `holditem`||cursoroffset|Vector3|The offset to apply to the cursor when selecting this actor|
|itemoffset|Vector3|The offset to render the item relative to the entity|

### Alive and death turn tracking

|Name|Type|Description|
|---:|---|---|
|turnssincedeath|int|The amount of main turns since death, 0 if the actor is alive|
|turnsalive|int|The amount of main turns since the battle started or their last death if any occured|

## Unused fields
These fields are never referenced or never used in any meaningful ways.

|Name|Type|Description|
|---:|---|---|
|tiredpart|ParticleSystem|UNUSED|
|noblock|bool|UNUSED|
|pointer|int|UNUSED, This field is maintained to be the same value as `partypointer`, but by indexing `playerdata` instead. This means that `playerdata[X].pointer` is the same than `partypointer[X]` where `X` is a valid `playerdata` index. In practice, this field is only written into making it unused|
|turnsnodamage|int|UNUSED|
|hardhp|int|UNUSED|
|harddef|int|UNUSED|
|lv|int|UNUSED|
|id|int|UNUSED (only read from [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md), but never written to so it stays at 0)|
|noexpatstart|bool|UNUSED (intended for enemy party members only, only read during damage pipeline, but never written to so it always have the value false)|
