# Areas

An `Areas` enum value represents a group of [Maps](../Maps.md) that belongs to a certain location in the game. Each map must belong to one `Areas` and they are important in determining the scope of any [Regionalflags](../../Flags%20arrays/Regionalflags.md). 

The current area is saved on the [Save File](../../External%20data%20format/Save%20File.md).

## Remarks about `librarystuff`

Each area is bound to a bool value stored in a 2 dimensional jagged array located at index 4 of the `MainManager.instance.librarystuff` array that indicates whether or not the area was seen. This allows the game to track which areas to show on the world map. The subarray of `librarystuff` contains 256 elements (most are not used as the array is overprosioned in size). 

The entire `librarystuff` two dimensional array is saved on the [Save File](../../External%20data%20format/Save%20File.md).

## Areas table
The following table shows all the `Areas` enum value and their English names for convenience.

|ID|Name|English Name|
|-:|----|------------|
|0|BugariaOutskirts|Bugaria Outskirts|
|1|BugariaCity|Ant Kingdom City|
|2|Snakemouth|Snakemouth Den|
|3|Desert|Lost Sands|
|4|GoldenHills|Golden Hills|
|5|GoldenWay|Golden Path|
|6|GoldenSettlement|Golden Settlement|
|7|BarrenLands|Forsaken Lands|
|8|FarGrasslands|Far Grasslands|
|9|WildGrasslands|Wild Swamplands|
|10|DefiantRoot|Defiant Root|
|11|SandCastle|Ancient Castle|
|12|Beehive|Bee Kingdom Hive|
|13|HoneyFactory|Honey Factory|
|14|RubberPrison|Rubber Prison|
|15|GiantLair|Giant's Lair|
|16|MetalLake|Metal Lake|
|17|MetalIsland|Metal Island|
|18|TermiteCity|Termite Capitol|
|19|WaspKingdom|Wasp Kingdom Hive|
|20|BanditHideout|Bandit Hideout|
|21|StreamMountain|Stream Mountain|
|22|ChomperCaves|Chomper Caves|
|23|FishingVillage|Fishing Village|
|24|UpperSnakemouth|Upper Snakemouth|
