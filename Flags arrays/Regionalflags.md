# Regionalflags

A `regionalflag` is a bool which is a variable the game can access globally while its value is kept until an `Area` change occurs in the game or until a reset is specifically requested by the game. When a reset occurs, all regionalflag are set to false. They are saved on the [Save File](../Data%20format/Save%20File.md) and are used to track temporary progress in an area such as defeating an enemy in the overworld. There are 100 regional flags available in the `MainManager.instance.regionalflags` array.

## Notes about the regionalflags tables

Since the meaning of the regional flags depends on the [Areas](../Enums%20and%20IDs/librarystuff/Areas.md), there will be one table per area in this document.

The instances in which the game specifically requests a reset will be mentioned in the table of the concerned area.

When an enemy is mentioned (which is most of the regional flags),
it is implied the value gets set to `true` when being defeated unless specified otherwise.

Any ID not specified implies UNUSED.

## Regional flags tables

### Bugaria Outskirts

|ID|Description|
|--|-----------|
|0|Rightmost Seedling at the map on the west of the Explorers Association|
|1|Second Seedling from the right at the map on the west of the Explorers Association|
|2|Leftmost Seedling from the right at the map on the west of the Explorers Association|
|4|Third Seedling from the right at the map on the west of the Explorers Association|
|5|Cutting the leftmost bush at the map to the east of the Snakemouth Den entrance|
|6|Seedling at the map to the east of the Snakemouth Den entrance|
|7|Getting the Honey Drop by cutting the leftmost bush at the map next to Chuck's place|
|10|Seedling in the middle (near the elevated platform) at the map the the south of the Explorers Association|
|11|Leftmost Seedling (near the tunnel entrance) at the map the the south of the Explorers Association|
|12|Rightmost (near the loading zone) Seedling at the map the the south of the Explorers Association|
|13|Getting a Crunchy leaf by cutting the second bush from the left at the map to the east of the Snakemouth Den entrance|
|15|Midge at the tunnel near Golden Path|
|16|Numbnail at the tunnel near Golden Path|
|17|Leftmost Underling at the map to the east of the Explorers Association|
|18|Rightmost Underling at the east map of the association|
|19|Seedling near the Cave Of Trials entrance|
|20|Seedling at the map next to Chuck's place|
|21|Rightmost Underling at the map to the west of Bugaria Piers|
|22|Cutting the leftmost bush at Seedling Heaven (it has a chance to drop a Tangy Berry)|
|23|Right Seedling at Seedling Heaven|
|24|Bottom Seedling at Seedling Heaven|
|25|Left Seedling at Seedling Heaven|
|26|Belostoss in the cave with the wind blowers|
|27|Leftmost Underling at the map to the west of Bugaria Piers|
|28|Ironnail in the cave with the wind blowers|
|50|Ironnail in the tunnel near Golden Path|
|51|Belostoss in the tunnel near Golden Path|

### Ant Kingdom City

|ID|Description|
|--|-----------|
|1|Talked to Fry once|

### Snakemouth Den

 > 
 > When reaching the Ancien Mask room, the regional flags are programmed to reset to false

|ID|Description|
|--|-----------|
|2|Depths, Zombiant in the center of the room located west of the Spider's room|
|3|Depths, leftmost Inichas in the room located west of the Spider's room|
|4|Depths, Inichas on the back right part in the room located west of the Spider's room|
|5|Depths, cutting the leftmost bush in the room located west of the Spider's room (drops a berry)|
|7|Depths, leftmost Zombiant in the room located west of the Spider's room|
|8|Depths, Jellyshroom on the back of the room located west of the Spider's room|
|9|Hit the coiled vine in the first room|
|10|Depths, got either an Aphid Egg, Honey drop or Crunchy Leaf by cutting the bush on the back left part of the room located west of the Spider's room|
|11|Depths, the closest Zombiant to the right exit of the room located west of the Spider's room|
|12|Depths, got the Magic Seed by cutting the rightmost bush in the room located east from the room with the doors|
|13|Depths, Inichas in the room located east from the room with the doors|
|14|Depths, Jellyshroom in the front of the mushrooms room|
|15|Depths, Jellyshroom in the back of the mushrooms room|
|17|Depths, Zombiant in the room located west from the room with the doors|
|18|Depths, Zombiant at the bottom of the room with the left door switch|
|19|Depths, Zombiant at the top of the room with the left door switch|
|20|Depths, Inichas in the room with the right door switch|
|21|Inichas in the second room (the one with the large door and the trap hole)|
|22|Rightmost Inichas in the first room|
|23|Leftmost Inichas in the first room|
|25|Depths, Inichas on the south part of the room with the doors|
|26|Depths, Zombiant on the right part of the room with the doors|
|27|Depths, Jellyshroom on the left part of the room with the doors|
|30|Depths, Jellyshroom in the room located east from the room with the doors|
|31|Depths, Jellyshroom in the room with the right door switch|
|32|Depths, got the red berry by cutting the rightmost bush in the room with the doors|

### Lost Sands

|ID|Description|
|--|-----------|
|0|Bandit at the map located northwest from the map with the blue fortk (south entrance)|
|1|Cactiling located south at the large sand pit map|
|2|Psicorp located south at the large sand pit map|
|3|Cactiling located east at the large sand pit map|
|4|Psicorp at the map with the blue fork (south entrance)|
|5|Cactiling located on the back of the map with the Bandit Hideout entrance|
|6|Cactiling located the closest of the Bandit Hideout entrance|
|7|Thief at the map located east from the Caravan shop|
|8|Arrow Worm at the map located north of the map with the blue fork (south entrance)|
|10|Arrow Worm at the map located west from the map where the Caravan was ambushed by bandits and Wasps|
|11|Cactiling at the map located east from Defiant Root|
|12|Psicorp at the map located east from Defiant Root|
|13|Thief at the large sand pit map|
|14|Bandit at the large sand pit map|
|15|Midge at the map located north of the map where the Golden Settlement is accessible from|
|16|Psicorp at the map located north of the map where the Golden Settlement is accessible from|
|17|Chomper at the map located north of the Golden Settlement|
|18|Cactiling at the map located north of the Golden Settlement|
|19|Bandit at the map with the blue fork (south entrance)|
|20|Burglar at the map located north of the map where the Golden Settlement is accessible from|
|21|Bandit at the map located east from the map where the Golden Settlement is accessible from|
|22|Thief at the map located south from the Caravan shop|
|23|Burglar at the map located east from the Caravan shop|
|24|Burglar at the map where the Caravan was ambushed by bandits and Wasps|
|25|Getting the red berry by cutting a bush on an elevated platform at the map located north of the Golden Settlement|
|50|Psicorp at the map located south from the Caravan shop|
|51|Arrow Worm at the map located south from the Caravan shop|
|52|Leftmost Cactiling at the map located west from the map with the blue fork (south entrance)|
|53|Rightmost Cactiling at the map located west from the map with the blue fork (south entrance)|
|54|Underling at the map located northwest from the map with the blue fortk (south entrance)|
|55|Psicorp at the map located northwest from the map with the blue fortk (south entrance)|
|56|Bandit at the map where the Caravan was ambushed by bandits and Wasps|
|58|Cactiling at the map located east from the map with the large sand pit|
|59|Thief at the map located east from the map with the large sand pit|
|60|Getting a Crunchy leaf by cutting the leftmost bush at the map located south from the Caravan shop|
|65|Getting a blue berry by cutting the bush on the front right of the left side of the map located west from the map where the Caravan was ambushed by bandits and wasps|
|66|Cutting the bush on the front left of the left side of the map located west from the map where the Caravan was ambushed by bandits and wasps|
|67|Getting a blue berry by cutting the bush on the back left of the left side of the map located west from the map where the Caravan was ambushed by bandits and wasps|
|68|Getting a red berry by cutting the bush on the back right of the left side of the map located west from the map where the Caravan was ambushed by bandits and wasps|

### Golden Hills

|ID|Description|
|--|-----------|
|0|Got a Magic Seed by cutting the bush next to the crank on the back right part of the map located west from the map with the right angle wooden spinning bridge|
|1|Weevil in the map located west from the map with the right angle wooden spinning bridge|
|2|Acornling in the map located west from the map with the right angle wooden spinning bridge|
|3|Chomper in the map located west from the map with the right angle wooden spinning bridge|
|4|Numbnail in map with the right angle wooden spinning bridge|
|5|Chomper in the map with the right angle wooden spinning bridge|
|6|Chomper in the map located east from the entrance|
|7|Got a Burly Berry by cutting the bush closest to the rightmost crank in the map located north from the map with the right angle wooden spinning bridge|
|8|MIdge in the map located north from the map with the right angle wooden spinning bridge|
|10|Numbnail in the map located north from the map with the right angle wooden spinning bridge|
|11|Acornling in the map located north from the map with the right angle wooden spinning bridge|
|60|Acornling in the map locatest east of the map with the offerings receptacle|
|61|Chomper in the map locatest east of the map with the offerings receptacle|

### Golden Path

 > 
 > The regional flags are programmed to reset to false when crossing either the loading zone north of the map
 > with the 2 cranks next to eachother or the loading zone back to it from the north map

|ID|Description|
|--|-----------|
|0|Got a Clear Water by cutting the bush closest to the green spring next to the sign in the first map from Bugaria Outskirts|
|1|Acornling in the map located east from the map with the shop|
|3|Weevil in the map located east from the map with the shop|
|4|Rightmost Acornling in the first map from Bugaria Outskirts|
|5|Numbnail in the map located east from the map with the shop|
|6|Leftmost Acornling in the first map from Bugaria Outskirts|
|7|Weevil in the map with 2 cranks next to eachother|
|11|Rightmost Numbnail in the map with the 2 cranks next to eachother|
|12|Numbnail in the map located north of the map with the 2 cranks next to eachother|
|13|Weevil in the map located north of the map with the 2 cranks next to eachother|
|14|Acornling in the map located north of the map with the 2 cranks next to eachother|
|15|Acornling in the map before the map where Devourer appears|
|16|Chomper in the map before the map where Devourer appears|
|17|Weevil in the map before the map where Devourer appears|
|30|Leftmost Numbnail in the map with 2 cranks next to eachother|

### Golden Settlement

|ID|Description|
|--|-----------|
|1|Talked to Kut once|

### Forsaken Lands

|ID|Description|
|--|-----------|
|0|Mimic Spider in the map with the cut CD|
|1|Ironnail in the map with a pencil pointing towards the correct path|
|2|Plumpling in the map with a pencil pointing towards the correct path|
|3|Ironnail in the map located west from the Bugaria Outskirts entrance|
|4|Got a Squash by cutting the bush near the north exit of the map with a pencil pointing towards the correct path|
|6|Ironnail in the map with several wind blowers|
|7|Got a Squash by cutting the rightmost bush in the map with several pumpkins|
|8|Mimic Spider in the map located south of the map where the Primal Weevil appears|
|9|Ironnail in the map located south of the map where the Primal Weevil appears|
|10|Got a Magic Seed by cutting the bush to the left of the fountain on an elevated platform in the fake Ant Village|
|11|Got a Squash by cutting the rightmost bush in the map located south of the map where the Primal Weevil appears|
|13|Mothfly in the map with the cut CD pointing towards the correct path|
|23|Mothfly in the map with several wind blowers|
|24|Mothfly Cluster in the map with several wind blowers|
|25|Mimic Spider in the map with several wind blowers|
|26|Leftmost Ironnail in the map with several pumpkins|
|27|Rightmost Ironnail in the map with several pumpkins|
|30|Mothfly Cluster in the map located west of the map with several wind blowers|
|31|Mothfly in the map located west of the map with several wind blowers|

### Far Grasslands

|ID|Description|
|--|-----------|
|0|Destroyed the boulder on the back of the map located south of the map with the Ant Mines tunnel access|
|1|Destroyed the boulder on the front of the map located south of the map with the Ant Mines tunnel access|
|3|Flowerling in the map located north of the south entrance|
|4|Jumping Spider in the map of the south entrance|
|5|Topmost Wild Chomper in the map located east of the south entrance|
|6|Mantidly in the map located west of the Fishing Village|
|7|Flowerling in the map located west of the south entrance|
|8|Jumping Spider in the map located west of the south entrance|
|9|Got a red berry by cutting the bush in the back left corner (back of the cluster of 3) in the map located west of the south entrance|
|10|Got a blue berry by cutting the bush in the back left corner (left of the cluster of 3) in the map located west of the south entrance|
|11|Got a blue berry by cutting the bush in the back left corner (front of the cluster of 3) in the map located west of the south entrance|
|12|Jumping Spider in the map located east of the south entrance|
|13|Jumping Spider in the map located west of the Fishing Village|
|14|Flowerling in the map located west of the Fishing Village|
|15|Wild Chomper in the map of the south entrance|
|16|Lowermost Wild Chomper in the map located east of the south entrance|
|17|Mantidly in the map located west of the south entrance|
|18|Mantidly in the map located south of the map where Yin evolves|
|19|Flowerling in the map located south of the map where Yin evolves|
|20|Wild Chomper in the map located south of the map with the Ant Mines tunnel access|
|21|Jumping Spider in the map located south of the map with the Ant Mines tunnel access|
|22|Wild Chomper in the map located south of the map where Yin evolves|
|23|Mantidly in the map located north of the south entrance|
|25|Wild Chomper in the map located north of the south entrance|

### Wild Swamplands

|ID|Description|
|--|-----------|
|0|Destroyed the boulder above a piece of wood in the map located south of the map with the broken wooden bridge|
|1|Destroyed the rightmost boulder of the map located south of the map with the broken wooden bridge|
|2|Madesphy in the south entrance|
|3|Destroyed the leftmost boulder of the map located south of the map with the broken wooden bridge|
|4|Got a red berry by cutting the closest bush to the green spring in the map located north from the south entrance|
|5|Got a blue berry by cutting the frontmost bush in the map located north from the south entrance|
|6|Got a Honey Drop by cutting the rightmost bush in the map located north from the south entrance|
|7|Madesphy in the map located north the south entrance|
|8|Leafbug Clubber in the map located west from the map with the broken wooden bridge|
|9|Leafbug Ninja in the map located west from the map with the broken wooden bridge|
|10|Destroyed the closest boulder of the Flowerling in the map located south of the map with the broken wooden bridge|
|11|Destroyed the leftmost boulder of the map with a lever surrounded by a spike pit|
|12|Destroyed the rightmost boulder of the map with a lever surrounded by a spike pit|
|13|Got a Magic Seed by digging in the dig spot in the map located east of the map with a lever surrounded by a spike pit|
|14|Wild Chomper in the map located north the south entrance|
|15|Leafbug Archer in the map located south of the map with the broken wooden bridge|
|16|Leafbug Ninja in the map located south of the map with the broken wooden bridge|
|17|Leafbug Clubber in the map with a lever surrounded by a spike pit|
|18|Madesphy in the map with a lever surrounded by a spike pit|
|20|Leafbug Archer in the map located south of the map with a lever surrounded by a spike pit|
|21|Madesphy in the map located south of the map with a lever surrounded by a spike pit|
|23|Leafbug Clubber in the map located east of the map with a lever surrounded by a spike pit|
|24|Madesphy in the map located east of the map with a lever surrounded by a spike pit|
|25|Leafbug Ninja in the map located east of the map with a lever surrounded by a spike pit|
|26|Rightmost Leafbug Archer in the map located east of the map with the broken wooden bridge|
|27|Madesphy in the map located east of the map with the broken wooden bridge|
|28|Leftmost Leafbug Archer (only set the first time deafeating it) in the map located east of the map with the broken wooden bridge|
|29|Leafbug Clubber in the map located east of the map with the broken wooden bridge|
|30|Wild Chomper in the map located south of the map with a lever surrounded by a spike pit|
|31|Mantidly in the map located north the south entrance|
|32|Leftmost Wild Chomper in the map with a lever surrounded by a spike pit|
|33|Rightmost Wild Chomper in the map with a lever surrounded by a spike pit|
|34|Got the Clear Bomb by hitting the coiled vine in the map with a lever surrounded by a spike pit|

### Defiant Root

|ID|Description|
|--|-----------|
|10|Talked to Crisbee once|

### Ancient Castle

|ID|Description|
|--|-----------|
|0|Krawler in the entrance room|
|1|Leftmost Warden in the room located east from the central room first floor|
|2|Rightmost Warden in the room located east from the central room first floor|
|3|Haunted Cloth in the room located east from the central room first floor|
|4|Warden in the room with 4 pleasure plates|
|5|Psicorp in the central room|
|6|Arrow Worm in the room located north from the central room first floor|
|7|Haunted Cloth in the room located north from the central room first floor|
|8|Arrow Worm in the room with the rolling rocks|
|9|Haunted Cloth in the room with the rolling rocks|
|10|Psicorp in the room with the rolling rocks (the rightmost hidden part)|
|11|Krawler in the room located west from the central room|
|12|Warden in the central room|
|13|Haunted Cloth in the room before the Ancient Half room|
|14|Krawler in the room before the Ancient Half room|
|15|Warden in the room with the rolling rocks|

### Bee Kingdom Hive

NO REGIONAL FLAGS ARE USED FOR THIS AREA

### Honey Factory

|ID|Description|
|--|-----------|
|0|Processing area, Security Turret in the first room|
|1|Processing area, Denmuki in the first room|
|2|Processing area, Bee-Boop in the room located east from the central room|
|3|Processing area, Denmuki in the room located east from the central room|
|5|Processing area, Bee-Boop in the central room|
|6|Processing area, Denmuki in the central room|
|7|Processing area, Security Turret in the central room|
|8|Processing area, Security Turret in the room located west from the central room top floor|
|9|Processing area, Bee-Boop in the first room|
|10|Processing area, Security Turret in the room located east from the central room|
|11|Processing area, Denmuki in the room located west from the center room top floor|
|12|Processing area, leftmost Bee-Boop in the room located north from the central room top floor|
|13|Processing area, rightmost Bee-Boop in the room located north from the central room top floor|
|14|Storage area, Abomihoney in the pit of the north room|
|15|Storage area, leftmost Abomihoney in the north room|
|16|Storage area, rightmost Abomihoney in the north room|
|17|Storage area, Denmuki in the large room with many stacked boxes|
|18|Storage area, Security Turret in the large room with many stacked boxes|
|19|Storage area, Bee-Boop in the south room|
|20|Storage area, got the Magic Seed in the south room|

### Rubber Prison

|ID|Description|
|--|-----------|
|0|Wasp Trooper in the library|
|1|Ruffian in a cage with a grey lever neat a save point|
|2|Burglar in the room located north of the room with Venus and some cages|
|3|Wasp Trooper on the lowest bridge outside|
|4|Wasp Scout on the bridget that leads to Giant's Lair outside|
|5|Ruffian on the bridge that leads to Giant's Lair outside|
|6|Wasp Scout on the bridge that leads to the warden office outside|
|7|Wasp Driller in the first room of the first floor|
|8|Wasp Bomber in the room located south of the room with Venus and some cages|
|9|Wasp Scout in the room with Venus and some cages|
|10|Wasp Trooper in the room with Venus and some cages|
|11|Wasp Trooper in the gym|
|12|Ruffian in the room with the prison cells and the shock platforms|
|13|Wasp Driller in the gym|
|14|Wasp Bomber in the room located north of the room with Venus and some cages|
|16|Ruffian in the room located north of the room with Venus and some cages|
|17|Wasp Bomber in the cafeteria|
|19|Wasp Trooper in the security room|
|22|Ruffian in the library|
|24|Wasp Driller in the warden office|

### Giant's Lair

|ID|Description|
|--|-----------|
|0|Krawler in the room above the oven roomm|
|1|Dead Lander α in the room located west from the refrigirator entrance|
|2|Dead Lander β in the room located west from the refrigirator entrance|
|3|Rightmost Dead Lander α in the room located east from the entrance|
|4|Dead Lander β in the room located east from the entrance|
|5|Leftmost Dead Lander α in the room located east from the entrance|
|6|Krawler in the top floor of the refrigirator|
|8|Krawler in the first floor of the refrigirator|
|9|Got the Mystery Berry in the room located west from the refrigirator entrance|
|10|Krawler near the rightmost red spring in the oven room|
|11|Leftmost Warden in the oven room|
|12|Rightmost Krawler in the oven room|
|13|Destroyed the boulder on the large book in the oven room|
|14|Rightmost Haunted Cloth in the oven room|
|15|Haunted Cloth behind the large book in the oven room|
|16|Rightmost Warden in the oven room|
|17|Haunted Cloth in the room above the oven room|
|18|Dead Lander α in the room above the oven room|
|19|Warden in the room above the oven room|
|20|Krawler near the leftmost red spring in the oven room|
|30|Got a Magic Seed by digging in the dig spot in the oven room|

### Metal Lake

|ID|Description|
|--|-----------|
|0|Water Strider at the bottom middle near a piece of wood|
|1|Wasp Scout near the green carton box|
|2|Wasp Scout closest to Rubber Prison|
|3|Water Strider closest to the Fishing village|
|4|Water Strider between Metal Island and Bugaria Piers (topmost one)|
|5|Water Strider closest to the Termite Capitol|
|6|Water Strider closest to the bottom right Island|
|7|Water Strider between Metal Island and Bugaria Piers (bottommost one)|

### Metal Island

NO REGIONAL FLAGS ARE USED FOR THIS AREA

### Termite Capitol

NO REGIONAL FLAGS ARE USED FOR THIS AREA

### Wasp Kingdom Hive

|ID|Description|
|--|-----------|
|0|Destroyed the boulder in the prison cells room|
|1|Wasp Trooper in the room with the blue save point|
|2|Wasp Trooper in the prison cells room|
|3|Wasp Scout in the prison cells room|

### Bandit Hideout

|ID|Description|
|--|-----------|
|0|Burglar in the first room from the Defiant Root well passage|
|1|Bandit in the first room from the Defiant Root well passage|
|2|Thief in the first room from the Defiant Root well passage|
|3|Burglar in the room with a table and a water droplet|
|4|Bandit in the room located south from the prison cells|
|5|Thief in the room located south from the prison cells|
|6|Got a Squash in the cafeteria near the cook|
|13|Thief in the cafeteria|
|14|Burglar in the cafeteria|

### Stream Mountain

|ID|Description|
|--|-----------|
|4|Belostoss in the room located east of the room where the Tydal Wyrm appears|
|5|Rightmost Water Strider in the room located east of the room where the Tydal Wyrm appears|
|6|Leftmost Water Strider in the room located east of the room where the Tydal Wyrm appears|
|8|Water Strider in the room locates east from the west entrance|
|9|Belostoss in the room locates east from the west entrance|
|10|Diving Spider in the room locates east from the west entrance|
|12|Water Strider in room with a crystal and exactly 2 tree branches|
|13|Diving Spider in room with a crystal and exactly 2 tree branches|
|14|Belostoss in the first room from the east entrance|

### Chomper Caves

|ID|Description|
|--|-----------|
|0|Chomper Brute in the lower part of the first room from the entrance|
|1|Chomper in the first room from the entrance|
|2|Middle Chomper in the room located west from the first room|
|4|Chomper Brute in the upper part of the room located west from the first room|
|3|Chomper Brute in the lower part of the room located west from the first room|
|5|Leftmost Chomper in the room located west from the first room|
|6|Rightmost Chomper in the room located west from the first room|
|7|Chomper Brute in the upper part of the first room from the entrance|

### Fishing Village

NO REGIONAL FLAGS ARE USED FOR THIS AREA

### Upper Snakemouth

|ID|Description|
|--|-----------|
|0|Got a Mushroom on top of the gears in the room located west from the center room|
|1|Bloatshroom in the first room from the entrance|
|2|Bloatshroom in the center room|
|3|Bloatshroom in the room located northwest from the center room|
|4|Bloatshroom in the room located southwest from the center room|
|5|Zombiant in the room located southwest from the center room|
|6|Zombeetle in the room located directly north from the center room|
|7|Zombee in the room located northwest from the center room|
|8|Zombeetle in the room located southeast from the center room|
|9|Zombee near a moving platform in the room located southeast from the center room|
|10|Bloatshroom in the room located southeast from the center room|
|11|Zombee in the center room|
|12|Zombee at the top of the room located southeast from the center room|
|13|Bloatshroom in the room located west from the center room|
