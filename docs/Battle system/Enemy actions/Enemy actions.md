# Enemy actions
This directory contains all the enemy actions logic contained in [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md). They are important and complex enough to warrant its own directory in the BattleControl documentation.

An enemy action is any piece of logic that is specific to the case where the actor sent to DoAction is an enemy party member. In that case, the actionid is interpreted as the `enemydata` index and the enemy party member's [enemy](../../Enums%20and%20IDs/Enemies.md) id determines the action logic that will be processed.

Anything else is detailed in the DoAction documentation. For the player actions, check the [player actions](../Player%20actions/Player%20actions.md) documentation.

Each enemy entry has one action, but its complexity allows them to perform many different tasks within that same action. Notably, all enemy actions is contained in a while(true) loop which allows them to reexecute as many times as needed before ending.

## Enemy actions table
Since the vast majority of the enemies in the game have their own action logic, their links will be listed per enemy id in the following table:

TODO: Add the links and files

|ID|Name|English Name|
|-:|----|------------|
|0|CordycepsAnt|Zombiant|
|1|Mushroom|Jellyshroom|
|2|Spuder|Spider|
|3|Zasp|Zasp|
|4|Cactus|Cactiling|
|5|Pseudoscorpion|Psicorp|
|6|Thief|Thief|
|7|Bandit|Bandit|
|8|ArmoredPoly|Inichas|
|9|Seedling|Seedling|
|10|FlyingSeedling|Flying Seedling|
|11|MakiTutorial|Maki|
|12|MothWeb|Web|
|13|SpuderReal|Spider|
|14|Sneil|Numbnail|
|15|Mothiva1|Mothiva|
|16|Acornling|Acornling|
|17|Weevil|Weevil|
|18|MrTester|Mr. Tester|
|19|AngryPlant|Venus' Bud|
|20|FlyTrap|Chomper|
|21|Acolyte|Acolyte Aria|
|22|AcolyteVine|Vine|
|23|Beetle|Kabbu|
|24|VenusBoss|Venus' Guardian|
|25|WaspTrooper|Wasp Trooper|
|26|WaspBomber|Wasp Bomber|
|27|WaspDriller|Wasp Driller|
|28|WaspHealer|Wasp Scout|
|29|Midge|Midge|
|30|Underling|Underling|
|31|Scarlet|Monsieur Scarlet|
|32|GoldenSeedling|Golden Seedling|
|33|Sandworm|Arrow Worm|
|34|Carmina|Carmina|
|35|SeedlingKing|Seedling King|
|36|MidgeBroodmother|Broodmother|
|37|Plumpling|Plumpling|
|38|Flowering|Flowerling|
|39|Burglar|Burglar|
|40|BanditLeader|Astotheles|
|41|MotherChomper|Mother Chomper|
|42|Ahoneynation|Ahoneynation|
|43|BeeBot|Bee-Boop|
|44|BeeTurret|Security Turret|
|45|ShockWorm|Denmuki|
|46|BeeBoss|Heavy Drone B-33|
|47|MenderBot|Mender|
|48|Abomihoney|Abomihoney|
|49|Scorpion|Dune Scorpion|
|50|SandWyrm|Tidal Wyrm|
|51|Kali|Kali|
|52|Zombee|Zombee|
|53|Zombeetle|Zombeetle|
|54|ZombieRoach|The Watcher|
|55|PeacockSpider|Peacock Spider|
|56|Bloatshroom|Bloatshroom|
|57|Krawler|Krawler|
|58|Cape|Haunted Cloth|
|59|SandWall|Sand Wall|
|60|IceWall|Ice Wall|
|61|CursedSkull|Warden|
|62|WaspKingIntermission|Wasp King|
|63|JumpingSpider|Jumping Spider|
|64|MimicSpider|Mimic Spider|
|65|LeafbugNinja|Leafbug Ninja|
|66|LeafbugArcher|Leafbug Archer|
|67|LeafbugClubber|Leafbug Clubber|
|68|SkullCaterpillar|Madesphy|
|69|Centipede|The Beast|
|70|ChomperBrute|Chomper Brute|
|71|Mantidfly|Mantidfly|
|72|WaspGeneral|General Ultimax|
|73|WildChomper|Wild Chomper|
|74|TermiteSoldier|Cross|
|75|TermiteNasute|Poi|
|76|PrimalWeevil|Primal Weevil|
|77|FalseMonarch|False Monarch|
|78|Mothfly|Mothfly|
|79|MothflyCluster|Mothfly Cluster|
|80|Ironclad|Ironnail|
|81|ToeBiter|Belostoss|
|82|Ruffian|Ruffian|
|83|Strider|Water Strider|
|84|DivingSpider|Diving Spider|
|85|Cenn|Cenn|
|86|Pisci|Pisci|
|87|DeadLanderA|Dead Lander α|
|88|DeadLanderB|Dead Lander β|
|89|DeadLanderG|Dead Lander γ|
|90|WaspKing|Wasp King|
|91|EverlastingKing|The Everlasting King|
|92|Maki|Maki|
|93|Kina|Kina|
|94|Yin|Yin|
|95|UltimaxTank|ULTIMAX Tank|
|96|Zommoth|Zommoth|
|97|Fisherman|Riz|
|98|Pitcher|Devourer|
|99|SandWyrmTail|Tail|
|100|PisciWall|Rock Wall|
|101|KeyR|Ancient Key|
|102|KeyL|Ancient Key|
|103|Tablet|Ancient Tablet|
|104|PitcherFlytrap|Flytrap|
|105|FireKrawler|FireKrawler|
|106|FireWarden|FireWarden|
|107|FireCape|FireCape|
|108|IceKrawler|IceKrawler|
|109|IceWarden|IceWarden|
|110|TANGYBUG|\|[glitchy](../../SetText/Individual%20commands/Glitchy.md),1\|TANGYBUG\|[glitchy](../../SetText/Individual%20commands/Glitchy.md)\||
|111|Stratos|Stratos|
|112|Delilah|Delilah|
