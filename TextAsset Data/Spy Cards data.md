# Spy Cards

Spy Cards data are split in 4 TextAssets in the game only loaded during the course of a CardGame (everything is handled independently by CardGame, not MainManager):

* By LoadCardData:
  * `Ressources/data/CardData`
  * `CardText` from the corresponding dialogue directory of the current [languageid](../SetText/languageid.md)
* By CardStart:
  * `Ressources/data/CardOrder`

The `CardData` is also loaded during the course of [Events](../Enums%20and%20IDs/Events.md) 210 (Spy Card tournament event).

## `CardData` data

The asset contains one line per Spy Card whose id corresponds to the line index. Each line contains fields separated by `,`:

|Name|Type|Description|
|----|----|-----------|
|tp|int|The cost of the card in TP|
|attack|int|The attack value of the card|
|enemyid|int|The [Enemies](../Enums%20and%20IDs/Enemies.md) id this card is bound to|
|namesizeX (unused)|float|The unused version of namesizeX|
|type|Type (int only)|The type of the card|
|effects|`@` separated list of effects|The effects of the card (see below for details)|
|tribe|`@` separated list of Tribe (int only)|The list of tribes this card belongs to|

The data will be loaded into `carddata[id]`, where `id` is the Spy Card id using the respective fields of a CardData struct except for namesizeX which is not loaded from here, it's loaded from `CardText`.

A Type can be one of the following values:

|Value|Name|
|-----|----|
|0|Attacker|
|1|Effect|
|2|Miniboss|
|3|Boss|

A Tribe can be one of the following values:

|Value|Name|
|-----|----|
|0|Seedling|
|1|Wasp|
|2|Fungi|
|3|Zombie|
|4|Plant|
|5|Bug|
|6|Machine|
|7|Thief|
|8|Unknown|
|9|Chomper|
|10|Leafbug|
|11|DeadLander|
|12|Mothfly|
|13|Spider|

A card effect contains 3 fields separated by `#`:

|Name|Type|Description|
|----|----|-----------|
|Effect|Effects (int only)|The specific effect this has|
|First parameter|int|The first parameter of the effect|
|Second parameter|int|The second parameter of the effect|

The first and second parameters's meaning depends on the effect. Here are the valid values for an Effects:

|Value|Name|
|-----|----|
|0|Attack|
|1|Defense|
|2|RandomMiniboss|
|3|Heal|
|4|HealIfWin|
|5|Summon|
|6|AttackOnCoin|
|7|DefenseOnCoin|
|8|SummonOnCoin|
|9|NumbFront|
|10|NumbFrontCoin|
|11|MultiplyPerTribe|
|12|AttackPerTribe|
|13|MultiplyAttackPerID|
|14|NumbAll|
|15|NumbAllCoin|
|16|DamageOnWin|
|17|AttackPerID|
|18|AttackIfOtherCard|
|19|HealIfOtherCard|
|20|AttackIfOpponentID|
|21|AttackIfOpponentTribe|
|22|MultiplyIfOpponentID|
|23|MultiplyIfOpponentTribe|
|24|DefenseIfOpponentID|
|25|DefenseIfOpponentTribe|
|26|IgnoreDefense|
|27|HealIfAttackAmmount|
|28|AttackPerOpponentTribe|
|29|AttackPerOpponentID|
|30|DefensePerOpponentTribe|
|31|DefensePerOpponentID|
|32|AttackOrDefenseCoin|
|33|HealIfWinOnce|
|34|AttackPerTribeOnce|
|35|AttackPerIDOnce|
|36|AttackOnce|
|37|Heal1OnTribeQuanity|
|38|NumbIfTribeAmmount|
|39|SummonRandomFromTribe|
|40|NumbIfOtherCard|
|41|DefenseOnOtherCard|
|42|AttackNextTurn|

TODO: Document the specific effects.

## `CardText` data

The asset contains one line per Spy Card whose id corresponds to the line index. Each line contains fields separated by `@`:

|Name|Type|Description|
|----|----|-----------|
|Description|[SetText](../SetText/SetText.md) string|The description of the card|
|namesizeX|float|The horizontal scaling to apply to the [SetText](../SetText/SetText.md) string of the [Enemies](../Enums%20and%20IDs/Enemies.md) name|

The data will be loaded into `carddata[id]`, where `id` is the Spy Card id using the respective fields of a CardData struct.

It should be noted that `namesizeX` is ignored when the [languageid](../SetText/languageid.md) is set to `Japanese`. Under that language, there is no scaling applied horizontally if the [Enemies](../Enums%20and%20IDs/Enemies.md) name is at most 3 characters. If the name is longer, then it's a Lerp from 0.45 to 1.0 where the value is 1 - the length of the name string / 10.0.

How the `namesizeX` work is normally, the [SetText](../SetText/SetText.md) call for the enemy name is sent with a size parameter of (0.5, 0.5), but the field is multiplied by the horizontal component. A value of 1.0 means 0.5 while a value of 0.5 means 0.25. It does not change the vertical component, only the horizontal one.

## `CardOrder` data

The asset contains the order to render the Spy Cards in the [Spy Cards List Type](../ItemList/List%20Types%20Group%20Details/Spy%20Cards%20List%20Type.md) where each line contains one field:

|Name|Type|Description|
|----|----|-----------|
|Spy Card id|int|The Spy Card id of the card to render at the position of the line index|

The data will be loaded into `order[i]` where i is the line index.
