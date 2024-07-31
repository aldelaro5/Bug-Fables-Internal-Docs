# Enemy actions
This directory contains all the enemy actions logic contained in [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md). They are important and complex enough to warrant its own directory in the BattleControl documentation.

An enemy action is any piece of logic that is specific to the case where the actor sent to DoAction is an enemy party member. In that case, the actionid is interpreted as the `enemydata` index and the enemy party member's [enemy](../../Enums%20and%20IDs/Enemies.md) id determines the action logic that will be processed.

Anything else is detailed in the DoAction documentation. For the player actions, check the [player actions](../Player%20actions/Player%20actions.md) documentation.

## How enemy actions work and how to read their documentation
This section can be served as a guide to the enemy action documentations linked below as it explains how an action is typically organised and how the documentation reflects that.

Each enemy entry has one action, but its complexity allows them to perform many different tasks within that same action. While each actions is free to do anything, they tend to follow a loosely defined structure that flows in this order:

- Pre move logic (optional)
- Move selection (mandatory)
- Move logic slated for usage (mandatory)
- Post move logic (optional)

### The move concept
A "move" in an enemy action page is loosely defined as a specific code path within the enemy action where they perform specific tasks at the expense of others. There isn't any concrete concept of a "move", but it exists loosely throughout the various enemy actions implementations. In fact, they don't have any names so each enemy documentations will indentify them by a number and a description that mostly reflects what the move does.

Each moves contains all the [damage pipeline](../BattleControl.md#damage-pipeline) and their high level logic. The logic doesn't contain everything, but it contains enough to identify the animations, audio and most importantly, the block timings from the player side. Each moves also contains notices about modifiers applied such as `nonphysical` and `dontusecharge`, see the [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md) documentation to learn more about them.

The decision of which move to use is often complex and it involves another key concept of enemy actions: the move selection process.

### The move selection process
An enemy action typically (there's some exceptions) don't operate deterministically with each other. They have a decision making process that frequently involves RNG or other more deterministic conditions. This process exists in various forms, but a very common one is generating a number using weighted RNG odds and use that number as a switch statement whose arms leads to the different moves's logic. This isn't always the case and the action might employ a mix of methods to make this decision, but the end goal is the same: there's always a decision process in which a move gets selected in the end. 

This process is what is reffered to as the "move selection" process. Every actions is assumed to have one in the documentation pages as the vast majority does have a process to determine this. It helps to reason that this concept exists even though there is nothing preventing an action from doing anything outside of this and there's no concept that defines this formally. The move selection also might not always have the same odds: enemies can decide to change the weights of them under certain conditions. In all cases, the documentation will always have a move selection section that documents this process.

### About rerolls and redirection
This selection process however isn't always simple as selecting a move and using it right away. Several enemies might not want to use certain moves under certain conditions. Due to the way the enemy actions are commonly done, the prefered method to implement this is rejection sampling: let the move selection process select a move, but falback if it's found that the move shouldn't be used.

This creates a distinction between when a move is SELECTED and when it is USED. The former might not always implies the latter as it's possible the falback is to use another move instead or to leverage the enemy action loop by issuing a continue directive to the enemy action loop.

### Enemy action loop
The enemy action loop is a special loop that encloses every enemy actions logic in a "while(true)" block. This isn't infinite since the loop always breaks out once the action is done, but it allows actions to issue a "continue" directive on it.

What this will do is restart the action logic without actually ending the action. It's essentially used as a more complex "goto" statement, but specific to enemy actions logic. The main usage of this feature is when a move is selected, but its usage was deemed to not fufill their requirements. Since enemy actions would rather prefer to use refjection sampling, issuing a continue directive allows to reroll the move selection process because it's something that's typically done very early in the action logic.

Not all actions employ this method (it might not always be feasible), but a lot do which is enough to mention it specifically as the enemy action loop.

### `data` slots
Every actor has a `data` field which is an int array, but it's exclusively reserved for enemy actions. It's a general purpose integer array to keep track of various information that the enemy needs to preserve accross multiple actor turns (as each actor turn is a separate DoAction call). It's an optional features: enemies don't have to use it, but it's there if they need it.

The enemy action is what decides the amount of slots to use if any and their entire semantics or valid values. Due to this, each documentation has a section about their usage if they are used. Typically, the initialisation of the field is done if it's not null as the starting value on the enemy's first actor turn is null.

### About [HardMode](../Damage%20pipeline/HardMode.md) changes
While HardMode already affects the [enemydata](../Actors%20states/Enemy%20features.md#difficulty-scaling) loading process to scale some stats with the difficulty, this isn't the only way enemies changes with it. Each action very frequently introduces a tailored set of changes if HardMode returns true. It's used so much that DoAction has a local declared called "hardmode" which is set to the return of HardMode which enemy actions logic can consult easily.

Due to how important these logic changes are in enemy actions with difficulty scaling, each enemies actions will have a dedicated section covering all the changes if HardMode returns true or any other relevant difficulty scaling information.

### About assumptions
Not all enemies are expected to function in all situations. There are enemies which are expected to operate in certain conditions, but there's nothing in the action logic that would enforce those condtions. In other words, they are assumed.

Enemy actions documentation will contain a section that details what is assumed and why the enemy wouldn't work properly if those assumptions aren't respected.

### About out of actions logic
Enemies might have specific logic done outside of DoAction. If that's the case, the enemy actions pages will contain sections to indicate them.

## Enemy actions table
Since the vast majority of the enemies in the game have their own action logic, their links will be listed per enemy id in the following table:

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
|101|[KeyR](Enemies/KeyR%20and%20KeyL.md)|Ancient Key||
|102|[KeyL](Enemies/KeyR%20and%20KeyL.md)|Ancient Key||
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
