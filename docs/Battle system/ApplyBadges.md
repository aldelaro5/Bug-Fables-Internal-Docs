# ApplyBadges
This method applies all the static [medals](../Enums%20and%20IDs/Medal.md) effects defined in their data for the whole player party. It belongs to MainManager.

It first starts by resetting instance.`speedup` (an UNUSED field) to true and instance.`maxtp` to instance.`basetp`.

ResetPlayerStats is called on player party member which resets fields to defaults:

|Field|Value set|
|----:|-----|
|`lockitems`|false|
|`lockskills`|false|
|`locktri`|false|
|`lockrelayreceive`|false|
|`maxhp`|`basehp`|
|`atk`|`baseatk`|
|`def`|`basedef`|
|`poisonres`|0|
|`sleepres`|0|
|`freezeres`|0|
|`numbres`|0|

After, all the `BadgeEffects` of every applicable medals are applied just as described in the [medal effects data documentation](../TextAsset%20Data/Medals%20data.md#medal-effects).

Finally, instance.`tp` is clamped from 0 to instance.`maxtp` and every player party member's `hp` is clamped from 0 to their `maxhp`.