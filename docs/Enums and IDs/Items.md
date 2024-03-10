# Items
An `Items` enum value represents an item in the game which can be collected in the overworld. The items in possessions are saved in the [Save File](../External%20data%20format/Save%20File.md) in 3 arrays corresponding to regular inventory, key items and stored items respectively. They are mapped directly to each item data entry.

## Items table
The following table shows all the `Items` enum value and their English names for convenience.

|ID|Name|English Name|
|-:|----|------------|
|0|CrunchyLeaf|Crunchy Leaf|
|1|HoneyDrop|Honey Drop|
|2|VitalitySeed|Spicy Berry|
|3|GenerousSeed|Burly Berry|
|4|VigorousSeed|Super Pepper|
|5|BurlySeed|Iron Seed|
|6|MoneySmall|RESERVED|
|7|MoneyMedium|RESERVED|
|8|Mistake|Mistake|
|9|CookedLeaf|Leaf Omelet|
|10|HoneydLeaf|Honey'd Leaf|
|11|MagicDrops|Magic Seed|
|12|ClearWater|Clear Water|
|13|Mushroom|Mushroom|
|14|CookedShroom|Cooked Shroom|
|15|GlazedShroom|Mushroom Salad|
|16|LeafSalad|Leaf Salad|
|17|AphidEgg|Aphid Egg|
|18|Omelet|Fried Egg|
|19|HeartyBreakfast|Hearty Breakfast|
|20|GlazedHoney|Glazed Honey|
|21|HoneyShroom|Sweet Shroom|
|22|HoneyDanger|Sweet Danger|
|23|HardSeed|Hard Seed|
|24|DotBall|Dotted Ball|
|25|GBugRangerPlushie|Bug Ranger Plushie|
|26|DangerShroom|Danger Shroom|
|27|ExplorerPermit|Explorer Permit|
|28|CookedDanger|Cooked Danger|
|29|SweetCrystal|Coal Crystal|
|30|SpicyBomb|Spicy Bomb|
|31|PoisonBomb|Poison Bomb|
|32|KingDinner|Queen's Dinner|
|33|MushroomStick|Mushroom Skewer|
|34|RoastBerry|Roasted Berries|
|35|Abomihoney|Abomihoney|
|36|ClearBomb|Clear Bomb|
|37|AntCompass|Ant Compass|
|38|ChomperSeed|Chomper Seed|
|39|BerryJuice|Berry Juice|
|40|NumbDart|Numbnail Dart|
|41|Map|Map of Bugaria|
|42|Ice|Magic Ice|
|43|Battery|Shock Berry|
|44|FrostBomb|Frost Bomb|
|45|NumbBomb|Numb Bomb|
|46|SleepBomb|Sleep Bomb|
|47|ShavedIce|Shaved Ice|
|48|AphidMilk|Aphid Dew|
|49|HoneyIceCream|Honey Ice Cream|
|50|HoneyMilk|Sweet Dew|
|51|IceCream|Ice Cream|
|52|LoreBook|Lore Book|
|53|FrozenSalad|Cold Salad|
|54|FlowerKey|Flower Key|
|55|OfferingA|Sun Offering|
|56|OfferingB|Moon Offering|
|57|MothivaDoll|Mothiva Doll|
|58|GHCrank|Wooden Crank|
|59|CrankHalfA|Big Crank Top Half|
|60|MainCrank|Big Crank|
|61|CrankHalfB|Big Crank Bottom Half|
|62|Abombhoney|Abombination|
|63|SpyData|Spy Data|
|64|PoisonSpud|Danger Spud|
|65|BakedYam|Baked Yam|
|66|FrenchFries|Spicy Fries|
|67|BurlyChips|Burly Chips|
|68|FlourBag|Bag of Flour|
|69|YamBread|Yam Bread|
|70|SpicyCandy|Spicy Candy|
|71|BurlyCandy|Burly Candy|
|72|DryBread|Dry Bread|
|73|NutCake|Nutty Cake|
|74|PoisonCake|Poison Cake|
|75|ShockCandy|Shock Candy|
|76|PlainTea|Plain Tea|
|77|TangyBerry|Tangy Berry|
|78|TangyJam|Tangy Jam|
|79|TangyJuice|Tangy Juice|
|80|SpicyTea|Spicy Tea|
|81|BurlyTea|Burly Tea|
|82|FrostPie|Frost Pie|
|83|HeartBerry|Heart Berry|
|84|BellBerry|Bond Berry|
|85|TangyPie|Tangy Pie|
|86|Donut|Crisbee Donut|
|87|TangyCarpaccio|Tangy Carpaccio|
|88|PoisonDart|Poison Dart|
|89|BedBug|Bed Bug|
|90|JellyBean|Succulent Berry|
|91|CookedJellyBean|Succulent Platter|
|92|DesertKey|Desert Key|
|93|QuestBook|Overdue Book|
|94|ChomperRibbon|Pretty Ribbon|
|95|BeeCard|Factory Pass|
|96|ShockShroom|Agaric Shroom|
|97|ShellOil|Shell Ointment|
|98|CrimsonOre|Crimson Ore|
|99|BeeHat|Bee Hat|
|100|BlankCard|Blank Card|
|101|SpadeCard|1-Suit Card|
|102|DualCard|2-Suit Card|
|103|TriadCard|3-Suit Card|
|104|FullCard|Full Suit Card|
|105|YinKey|Heaven Key|
|106|YangKey|Earth Key|
|107|LonglegSummoner|Longleg Summoner|
|108|StolenSilk|Stolen Silk|
|109|TrialKey|Mysterious Piece|
|110|GameToken|Game Tokens|
|111|HideoutKey|Rusty Key|
|112|TanjerinHorn|Orange Horn|
|113|SandCastleKey|Sand Castle Key|
|114|SandCastleSmallKey|Ancient Key|
|115|SandCastleBossKey|Big Ancient Key|
|116|SnakemouthKey|Peculiar Gem|
|117|BombyHat|Top Hat|
|118|SeedlingCrystal|Crystal Fruit|
|119|WaspKey|Wasp Key|
|120|JaydeStew|Jayde's Stew|
|121|BlackCherry|Dark Cherries|
|122|CherryPie|Cherry Pie|
|123|MiracleShake|Miracle Shake|
|124|BerrySmoothie|Berry Smoothie|
|125|Squash|Squash|
|126|SquashCandy|Squash Candy|
|127|SophiePetal|Sophie Petal|
|128|SucculentCookie|Succulent Cookies|
|129|Pudding|Sweet Pudding|
|130|SquashSoda|Mega-Rush™|
|131|HPPotion|HP Potion|
|132|ATKPotion|ATK Potion|
|133|DEFPotion|DEF Potion|
|134|TPPotion|TP Potion|
|135|SuperHPPotion|Super HP Potion|
|136|SuperTPPotion|Super TP Potion|
|137|MPPotion|MP Potion|
|138|ShadyNote|Shady Note|
|139|BlackPaint|Blackest Paint|
|140|RedPaint|Red Paint|
|141|DefRootCloak|Root Cloth|
|142|AntToy|Ant Doll|
|143|MIToy|Eastern Doll|
|144|MushroomCandy|Mushroom Gummies|
|145|TermiteLunch|Wrapped Lunch|
|146|CrownCrystal|Crystal Crown|
|147|DrowsyCake|Drowsy Cake|
|148|LeafCroisant|Leaf Croissant|
|149|Package|Package|
|150|RainbowCrystal|Crystal Feather|
|151|FangCrystal|Crystal Fang|
|152|CherryBomb|Cherry Bombs|
|153|SquashPuree|Squash Puree|
|154|MiteBurg|Mite Burger|
|155|ProteinShake|Aphid Shake|
|156|Guarana|Mystery Berry|
|157|SmallGear|Small Gear|
|158|MedGear|Medium Gear|
|159|BigGear|Big Gear|
|160|LabCard|Lab Card|
|161|PrisonKey|Prison Key|
|162|HustleSeed|Hustle Berry|
|163|PrisonBookA|History Book|
|164|PrisonBookB|History Book|
|165|PrisonBookC|History Book|
|166|PrisonBookD|History Book|
|167|LeafUmbrella|Leaf Umbrella|
|168|PoisonRibbon|Venom Ribbon|
|169|NumbRibbon|Shocking Ribbon|
|170|SleepRibbon|Drowsy Ribbon|
|171|BigMistake|Big Mistake|
|172|BurlyBomb|Burly Bomb|
|173|BerryShake|Berry Jam|
|174|BadBook|Bad Book|
|175|MechArm|Mechanical Claw|
|176|PlatinumCard|Platinum Card|
|177|Coffee|Hot Drink|
|178|CoffeeCandy|Hustle Candy|
|179|SquashPie|Squash Tart|
|180|PlumplingPie|Plumpling Pie|
|181|DangerDish|Danger Dish|
|182|HoneyPancake|Honey Pancakes|
|183|FlameRock|Flame Rock|
|184|CardTrophy|Card Trophy|
|185|RedRibbon|Team Ribbon|
|186|MoneyBig|RESERVED|