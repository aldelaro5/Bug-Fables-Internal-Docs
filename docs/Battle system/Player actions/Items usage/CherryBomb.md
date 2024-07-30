# `CherryBomb`
This item is a toss item and as such, parts of its logic are based off the [PeebleToss](../Skills/PeebleToss.md) action.

It's a special item action because `selecteditem` will get overriden to be one of the following [item](../../../Enums%20and%20IDs/Items.md) ids with uniform probabilities:

- `SpicyBomb`
- `PoisonBomb`
- `FrostBomb`
- `NumbBomb`
- `SleepBomb`
- `BurlyBomb`

The logic performed is the same than the new value of `selecteditem` with one exception: every [DoDamage](../../Damage%20pipeline/DoDamage.md) calls will have its damageammount increased by 2 compared to the normal call.

The rest is the same than if it was the actual overriden item.

There is some dead logic specifically related to this item however and it only contains the following [DoDamage](../../Damage%20pipeline/DoDamage.md) call, but it never happens under normal gameplay:

|Conditions|attacker|target|damageammount|property|overrides|block|
|---|---|---|---|---|---|---|
|Done for each enemy party member other than the `target` whose position + `freezeoffset` + their `height` in y has a square distance off the object after movement less than 15.5|null|The enemy party member|5|null|empty array|false|