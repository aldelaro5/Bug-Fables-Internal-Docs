# Enemy actions
This directory contains all the enemy actions logic contained in [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md). They are important and complex enough to warrant its own directory in the BattleControl documentation.

An enemy action is any piece of logic that is specific to the case where the actor sent to DoAction is an enemy party member. In that case, the actionid is interpreted as the `enemydata` index and the enemy party member's [enemy](../../Enums%20and%20IDs/Enemies.md) id determines the action logic that will be processed.

Anything else is detailed in the DoAction documentation. For the player actions, check the [player actions](../Player%20actions/Player%20actions.md) documentation.

Each enemy entry has one action, but its complexity allows them to perform many different tasks within that same action. Notably, all enemy actions is contained in a while(true) loop which allows them to reexecute as many times as needed before ending.

## Enemy actions table
Since the vast majority of the enemies in the game have their own action logic, their links will be listed per enemy id in the following table:

TODO: Add the links and files

|ID|Name|English Name|Notes|
|-:|----|------------|-----|
|0|[CordycepsAnt](Enemies/CordycepsAnt.md)|Zombiant||
|1|[Mushroom](Enemies/Mushroom.md)|Jellyshroom||
|2|[Spuder](Enemies/Spuder.md)|Spider||
|3|[Zasp](Enemies/Zasp.md)|Zasp||
|4|[Cactus](Enemies/Cactus.md)|Cactiling||
|5|[Pseudoscorpion](Enemies/Pseudoscorpion.md)|Psicorp||
|6|[Thief](Enemies/Thief.md)|Thief||
|7|[Bandit](Enemies/Bandit.md)|Bandit||
|8|[ArmoredPoly](Enemies/ArmoredPoly.md)|Inichas||
|9|[Seedling](Enemies/Seedling.md)|Seedling||
|10|FlyingSeedling|Flying Seedling|This enemy has no action logic|
|11|[MakiTutorial](Enemies/MakiTutorial.md)|Maki||
|12|MothWeb|Web|This enemy has no action logic|
|13|SpuderReal|Spider|This enemy has no action logic|
|14|[Sneil](Enemies/Sneil.md)|Numbnail||
|15|[Mothiva1](Enemies/Mothiva1.md)|Mothiva||
|16|[Acornling](Enemies/Acornling.md)|Acornling||
|17|[Weevil](Enemies/Weevil.md)|Weevil||
|18|[MrTester](Enemies/MrTester.md)|Mr. Tester||
|19|[AngryPlant](Enemies/AngryPlant.md)|Venus' Bud||
|20|[FlyTrap](Enemies/FlyTrap.md)|Chomper||
|21|[Acolyte](Enemies/Acolyte.md)|Acolyte Aria||
|22|AcolyteVine|Vine|This enemy has no action logic|
|23|[Beetle](Enemies/Beetle.md)|Kabbu||
|24|[VenusBoss](Enemies/VenusBoss.md)|Venus' Guardian||
|25|[WaspTrooper](Enemies/WaspTrooper.md)|Wasp Trooper||
|26|[WaspBomber](Enemies/WaspBomber.md)|Wasp Bomber||
|27|[WaspDriller](Enemies/WaspDriller.md)|Wasp Driller||
|28|[WaspHealer](Enemies/WaspHealer.md)|Wasp Scout||
|29|[Midge](Enemies/Midge.md)|Midge||
|30|[Underling](Enemies/Underling.md)|Underling||
|31|[Scarlet](Enemies/Scarlet.md)|Monsieur Scarlet||
|32|[GoldenSeedling](Enemies/GoldenSeedling.md)|Golden Seedling||
|33|[Sandworm](Enemies/Sandworm.md)|Arrow Worm||
|34|[Carmina](Enemies/Carmina.md)|Carmina||
|35|[SeedlingKing](Enemies/SeedlingKing.md)|Seedling King||
|36|[MidgeBroodmother](Enemies/MidgeBroodmother.md)|Broodmother||
|37|[Plumpling](Enemies/Plumpling.md)|Plumpling||
|38|[Flowering](Enemies/Flowering.md)|Flowerling||
|39|[Burglar](Enemies/Burglar.md)|Burglar||
|40|[BanditLeader](Enemies/BanditLeader.md)|Astotheles||
|41|[MotherChomper](Enemies/MotherChomper.md)|Mother Chomper||
|42|[Ahoneynation](Enemies/Ahoneynation.md)|Ahoneynation||
|43|[BeeBot](Enemies/BeeBot.md)|Bee-Boop||
|44|[BeeTurret](Enemies/BeeTurret.md)|Security Turret||
|45|[ShockWorm](Enemies/ShockWorm.md)|Denmuki||
|46|[BeeBoss](Enemies/BeeBoss.md)|Heavy Drone B-33||
|47|[MenderBot](Enemies/MenderBot.md)|Mender||
|48|[Abomihoney](Enemies/Abomihoney.md)|Abomihoney||
|49|[Scorpion](Enemies/Scorpion.md)|Dune Scorpion||
|50|[SandWyrm](Enemies/SandWyrm.md)|Tidal Wyrm||
|51|[Kali](Enemies/Kali.md)|Kali||
|52|[Zombee](Enemies/Zombee.md)|Zombee||
|53|[Zombeetle](Enemies/Zombeetle.md)|Zombeetle||
|54|[ZombieRoach](Enemies/ZombieRoach.md)|The Watcher||
|55|[PeacockSpider](Enemies/PeacockSpider.md)|Peacock Spider||
|56|[Bloatshroom](Enemies/Bloatshroom.md)|Bloatshroom||
|57|[Krawler](Enemies/Krawler.md)|Krawler||
|58|[Cape](Enemies/Cape.md)|Haunted Cloth||
|59|[SandWall](Enemies/SandWall.md)|Sand Wall||
|60|[IceWall](Enemies/IceWall.md)|Ice Wall||
|61|[CursedSkull](Enemies/CursedSkull.md)|Warden||
|62|[WaspKingIntermission](Enemies/WaspKingIntermission.md)|Wasp King||
|63|[JumpingSpider](Enemies/JumpingSpider.md)|Jumping Spider||
|64|[MimicSpider](Enemies/MimicSpider.md)|Mimic Spider||
|65|[LeafbugNinja](Enemies/LeafbugNinja.md)|Leafbug Ninja||
|66|[LeafbugArcher](Enemies/LeafbugArcher.md)|Leafbug Archer||
|67|[LeafbugClubber](Enemies/LeafbugClubber.md)|Leafbug Clubber||
|68|[SkullCaterpillar](Enemies/SkullCaterpillar.md)|Madesphy||
|69|[Centipede](Enemies/Centipede.md)|The Beast||
|70|[ChomperBrute](Enemies/ChomperBrute.md)|Chomper Brute||
|71|[Mantidfly](Enemies/Mantidfly.md)|Mantidfly||
|72|[WaspGeneral](Enemies/WaspGeneral.md)|General Ultimax||
|73|[WildChomper](Enemies/WildChomper.md)|Wild Chomper||
|74|[TermiteSoldier](Enemies/TermiteSoldier.md)|Cross||
|75|[TermiteNasute](Enemies/TermiteNasute.md)|Poi||
|76|[PrimalWeevil](Enemies/PrimalWeevil.md)|Primal Weevil||
|77|[FalseMonarch](Enemies/FalseMonarch.md)|False Monarch||
|78|[Mothfly](Enemies/Mothfly.md)|Mothfly||
|79|[MothflyCluster](Enemies/MothflyCluster.md)|Mothfly Cluster||
|80|[Ironclad](Enemies/Ironclad.md)|Ironnail||
|81|[ToeBiter](Enemies/ToeBiter.md)|Belostoss||
|82|[Ruffian](Enemies/Ruffian.md)|Ruffian||
|83|[Strider](Enemies/Strider.md)|Water Strider||
|84|[DivingSpider](Enemies/DivingSpider.md)|Diving Spider||
|85|[Cenn](Enemies/Cenn.md)|Cenn||
|86|[Pisci](Enemies/Pisci.md)|Pisci||
|87|[DeadLanderA](Enemies/DeadLanderA.md)|Dead Lander α||
|88|[DeadLanderB](Enemies/DeadLanderB.md)|Dead Lander β||
|89|[DeadLanderG](Enemies/DeadLanderG.md)|Dead Lander γ||
|90|[WaspKing](Enemies/WaspKing.md)|Wasp King||
|91|[EverlastingKing](Enemies/EverlastingKing.md)|The Everlasting King||
|92|[Maki](Enemies/Maki.md)|Maki||
|93|[Kina](Enemies/Kina.md)|Kina||
|94|[Yin](Enemies/Yin.md)|Yin||
|95|[UltimaxTank](Enemies/UltimaxTank.md)|ULTIMAX Tank||
|96|[Zommoth](Enemies/Zommoth.md)|Zommoth||
|97|[Fisherman](Enemies/Fisherman.md)|Riz||
|98|[Pitcher](Enemies/Pitcher.md)|Devourer||
|99|[SandWyrmTail](Enemies/SandWyrmTail.md)|Tail||
|100|PisciWall|Rock Wall|This enemy has no action logic|
|101|[KeyR](Enemies/KeyR.md)|Ancient Key||
|102|[KeyL](Enemies/KeyL.md)|Ancient Key||
|103|[Tablet](Enemies/Tablet.md)|Ancient Tablet||
|104|[PitcherFlytrap](Enemies/PitcherFlytrap.md)|Flytrap||
|105|FireKrawler|FireKrawler|This enemy has no action logic|
|106|FireWarden|FireWarden|This enemy has no action logic|
|107|FireCape|FireCape|This enemy has no action logic|
|108|IceKrawler|IceKrawler|This enemy has no action logic|
|109|IceWarden|IceWarden|This enemy has no action logic|
|110|[TANGYBUG](Enemies/TANGYBUG.md)|\|[glitchy](../../SetText/Individual%20commands/Glitchy.md),1\|TANGYBUG\|[glitchy](../../SetText/Individual%20commands/Glitchy.md)\|||
|111|[Stratos](Enemies/Stratos.md)|Stratos||
|112|[Delilah](Enemies/Delilah.md)|Delilah||
