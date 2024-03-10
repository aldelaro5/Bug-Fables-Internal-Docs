# BoardQuests
A `BoardQuests` is a task to be done in the game. While most of them are optional, each chapters have one quest reserved to them which are mandatory. The quests are managed by the quests board system which tells which quests are accessible, open, taken or completed. The state of the quest board are saved on the [Save File](../External%20data%20format/Save%20File.md). They are directly mapped to the quest data entry.

## BoardQuests table
The following table shows all the `BoardQuest` enum value and their English names for convenience.

|ID|Name|English Name|
|-:|----|------------|
|0|None|No Quests|
|1|InnQuest|Inn Review Required|
|2|ChuckQuest|I Want a New Taste|
|3|TheaterQuest|Theater Help Wanted!|
|4|ToyQuest|Lost Toy|
|5|LadybugQuest|My Brother is Gone!|
|6|UndergroundBar|Requesting Assistance|
|7|CableCar|Cable Car Bodyguard|
|8|SeedlingKing|Bounty: Seedling King|
|9|FalseMonarch|Bounty: False Monarch|
|10|MotherChomper|Bounty: Devourer|
|11|Prologue|Ch. 1: A Dysfunctional Trio|
|12|Chapter1|Ch. 2: Sacred Golden Hills!|
|13|Chapter2|Ch. 3: Factory Inspection|
|14|Chapter3|Ch. 4: Mysterious Lost Sands|
|15|Chapter4|Ch. 5: The Far Wildlands|
|16|Chapter5|Ch. 6: Assault at Rubber Prison|
|17|Chapter6|Ch. 7: The Everlasting Sapling|
|18|Crisbee|I Wanna Get Better!|
|19|Kut|My Specialty|
|20|Fry|A Smiling Dish|
|21|Sandwyrm|Bounty: Tidal Wyrm|
|22|Butomo|Ore Wanted|
|23|PeacockSpider|Bounty: Peacock Spider|
|24|Tanjerin|Huuuuuuuuuu...!!!|
|25|ZaspDoll|Lost Item|
|26|Leif|Leif's Request|
|27|LibraryantRed|Lost Books|
|28|Madeleine1|Butler Missing!|
|29|Venus|Team Snakemouth...|
|30|Bee|Vi's Request|
|31|PowerPlant|Power Plant Investigation|
|32|CardGame|Card Masters of Bugaria|
|33|CicadaBook|Book Return!|
|34|Vivi|I Want a Souvenir...|
|35|MenderQuest|Helpers Needed At Once!|
|36|Kali|Stolen Item|
|37|Isau|Rare Item Wanted!|
|38|Bomby|Dropped my Hat!|
|39|Madeleine2|Butler Missing Again!|
|40|GenEri|Explorer Check!|
|41|Mun|Help Me Get it Back!|
|42|BanditHunt|Bandit Hunt|
|43|ArtBee|In Search of Paint...|
|44|TermiteLunch|Lunch Delivery!|
|45|Alex|It's Time...!|
|46|Mayor|Want to Relive Memories...|
|47|SeedlingHunt|Seedling Hunt|
|48|Layna|Best Friend In The Fog!|
|49|Eetl|Parts Delivery|
|50|Farmer|Hydration Crisis!|
|51|RizSis|Sweets from Outside!|
|52|Wizard|Find The Ingredients!|
|53|Eremi|It's Too Hot!|
|54|MoleCricket|In Search of Something|
|55|Maki|Confidential|
|56|BadBook|Awful's Beauty|
|57|WaspTwins|They Took Her...!|
|58|BlacksmithGuy|My Mecha Claw!|
|59|WorkerTermite|Can't Sleep...!|
|60|Beetle|Kabbu's Request|
|61|Rebecca|Loose Ends|
|62|Roach|A New Hope|
|63|StratosDelilah|Getting Bored|
