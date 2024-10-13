# Discovery entry

A discovery entry is a collectible bound to a bool that tells whether or not the specific entry for the discovery is unlocked in the library. Their values are saved in the [Save File](../../External%20data%20format/Save%20File.md) and the array of them is located at index 0 of the `MainManager.instance.librarystuff` 2 dimensional jagged array and it contains 256 elements (most are not used as the array is overprosioned in size).

The entire `librarystuff` two dimensional array is saved on the [Save File](../../External%20data%20format/Save%20File.md).

## Discovery entries table
The following table shows all discovery entries with their English names for convenience.

NOTE: The ID is the actual internal index of `librarystuff`, but it isn't necessarily the displayed ID in game. For that, consult the [discoveryorder](../../TextAsset%20Data/Discoveries%20data.md#discoveryorder-data) which gives the displayed order by internal index.

|ID|English Name|
|-:|------------|
|0|Snakemouth Den|
|1|Snakemouth Depths|
|2|Explorer's Message|
|3|The Roaches' Village?|
|4|Inn Foremothers|
|5|The Settler Statue|
|6|The Ants|
|7|The Bees|
|8|The Termites|
|9|The Wasps|
|10|Queen Elizant I|
|11|The Anthill Palace|
|12|Aphids and Cochinaels|
|13|The Golden Festival|
|14|The Goddess' Statue|
|15|Honoured Employee|
|16|Sand Castle Mural|
|17|Snakemouth's Lab|
|18|Healing Sophie|
|19|Seedling Haven|
|20|Underground Tavern|
|21|The Ant Mines|
|22|Chomper Cavern|
|23|The Power Plant|
|24|Termite Statue|
|25|The Bee Kingdom|
|26|Balcony Telescope|
|27|B.O.S.S.|
|28|Defiant Root|
|29|Relic Museum|
|30|The Lost Sands|
|31|Tardigrade Idol|
|32|Bandit Hideout|
|33|Stream Mountain|
|34|Far Grasslands|
|35|Fishing Village|
|36|Wizard's Tower|
|37|Far Swamplands|
|38|Wasp Kingdom|
|39|Forsaken Lands|
|40|Ancient City|
|41|Termite Kingdom|
|42|Termacade|
|43|Termite Coliseum|
|44|Metal Island|
|45|Spy Card|
|46|Peacock Island|
|47|Rubber Prison|
|48|Giant's Lair|
|49|Sailor's Statue|
