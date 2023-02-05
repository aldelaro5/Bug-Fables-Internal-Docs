# Cave Of Trials data

The game contains a TextAsset located at `Ressources/data/Trials` from the root of the assets tree that contains data about Cave of Trials. It is only loaded during the course of [Event](../SetText/Commands/Individual%20commands/Event.md) 156 (Cave of Trials event) to gather the enemies ids used to start the battle for each round using StartBattle directly.

The asset contains a list of battles of a standard run in order, one for each line. Each lines contains one field:
Name | Type | Description
------ | ------- | --------
Enemy list | `,` separated list of [Enemies](../Enums%20and%20IDs/Enemies.md) id | The list of [Enemies](../Enums%20and%20IDs/Enemies.md) for the fight

## About the amount of fights

Interestingly, while the handling of this data is meant to function regardless of the number of lines in the asset, it is not meant to go above 50. This is because while the game doesn't specifically limit the number of battles to 50, the rewards [Giveitem](../SetText/Commands/Individual%20commands/Giveitem.md)'s `itemid` array, their corresponding [flags](../Flags%20arrays/flags.md) slot and their corresponding [Giveitem](../SetText/Commands/Individual%20commands/Giveitem.md)'s `type` are hardcoded to each contain 5 elements and are only given every 10 fights (called a fight block in the section below). Going beyond that would cause an exception to be thrown because there are no 6th element in each of the array.

It is worth noting that this is the only reason the amount of battles are limited to 50. Even the dialogue lines in the `CaveOfTrials` [Maps](../Enums%20and%20IDs/Maps.md) contains special strings (`@VAR@` and `@VAR2@`) that are replaced during the event by the appropriate number (current battle, amount of all battles in the run and amount of battles left).

## Random mode

Random mode still loads this asset, but the difference is that the actual battles are a randomized set of enemies. This implies that the amount of fights is still dependent on this asset rather than 50.

## Hardcoded rewards data

Here are the hardcoded rewards data present in the event:
Fight block | GiveItem's id | GiveItem's type | Reward description | flags slot
-------- | -------- | --------- | -------- | ------
0 | 77 | 0 | Tangy Berry | 503 
1 | 121 | 0 | Dark Cherries | 504
2 | 50 | 2 | Defense Exchange | 505
3 | 25 | 2 | TP Saver | 506
4 | 185 | 1 | Team Ribbon | 673
