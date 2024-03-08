# ApplyStatBonus
This method's purpose is to recalculate all player party member's stats based on all the bonuses that was accumulated so far. It belongs to MainManager.

It starts by calling ResetStats which resets all the stats to their starting value which are the following:

- `basehp`: 7 except for the `Beetle` [animid](../Enums%20and%20IDs/AnimIDs.md) where it's 9
- `baseatk`: 2
- `basedef`: 0
- Additionally, it also resets instance.`basetp` to 10

From there, the method only does anything if there's any `statbonus` (NOTE: This means the ApplyBadges call at the end is also skipped if there's none).

The method will then go through each `statbonus` and apply them as described in [stat bonus save file line](../External%20data%20format/Save%20File.md#line-10-array-line-stats-bonuses) by using the following fields (all player party members are impacted by the bonus if it targets the party):

- `basehp` for HP bonuses
- `baseatk` for attack bonuses
- `basedef` for defense bonuses
- instance.`basetp` for TP bonuses

After this, [ApplyBadges](ApplyBadges.md) is called.
