# Areas

An `Areas` enum value represents a group of [Maps](../Maps.md) that belongs to a certain location in the game. Each map must belong to one `Areas` and they are important in determining the scope of any [Regionalflag](../../Flags%20arrays/Regionalflag.md). 

The current area is saved on the [Save File](../../Data%20format/Save%20File.md).

## Remarks about `librarystuff`

Each area is bound to a bool value stored in an array located at index 0 of the `MainManager.instance.librarystuff` array that indicates whether or not the area was seen. This allows the game to track which areas to show on the world map. The subarray of `librarystuff` contains 256 elements (most are not used as the array is overprosioned in size). 

The entire `librarystuff` two dimensional array is saved on the [Save File](../../Data%20format/Save%20File.md).

## Areas table

|ID|Name|
|--|----|
|0|Bugaria Outskirts|
|1|Ant Kingdom City|
|2|Snakemouth Den|
|3|Lost Sands|
|4|Golden Hills|
|5|Golden Path|
|6|Golden Settlement|
|7|Forsaken Lands|
|8|Far Grasslands|
|9|Wild Swamplands|
|10|Defiant Root|
|11|Ancient Castle|
|12|Bee Kingdom Hive|
|13|Honey Factory|
|14|Rubber Prison|
|15|Giant's Lair|
|16|Metal Lake|
|17|Metal Island|
|18|Termite Capitol|
|19|Wasp Kingdom Hive|
|20|Bandit Hideout|
|21|Stream Mountain|
|22|Chomper Caves|
|23|Fishing Village|
|24|Upper Snakemouth|
