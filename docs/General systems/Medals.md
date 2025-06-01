# Medals
A medal (often internally refered to as a "badge") is piece of equipable gear to the player party or a specific player party member. They are defined in the game using various pieces of TextAsset documented in the medals data documentation page and are uniquely identified by a [medal id](../Enums%20and%20IDs/Medal.md) Medals are notable because they are a central concept in the game referenced by many other data or systems.

## Equip system
At the simplest level, a medal is composed of:

- A [medal id](../Enums%20and%20IDs/Medal.md)
- An equip state int value:
    - -2: Unequipped
    - -1: Equipped to the party
    - Anything else: Equipped to the party member whose `trueid` is the state value

Medals can be equipped or unequipped through the PauseMenu when interfacing with the medals listtype or the equipped medals listtype. A medal only be equipped if the MP cost of the medal (defined in the TextAsset mentioned above) satisfies the amount of remaining MP the party has. The MP capacity the party has is stored in `maxbp` and the amount remaining for use is stored in `bp`.

The array of medals on hands is `badges` and this array ends up being saved to the save file. This is what allows the game to keep track of what medals the player can equip and which ones are actually equipped. This doesn't take into account the MP cost because whether or not the equip is allowed is managed by PauseMenu.Update.

There are several utility methods that allows to deal with this equip system. Here's all of them.

```cs
public static void AddBadge(int id)
```
Add a `badges` element where the medal id is `id` and the equip state is -2 (unequipped).

```cs
public static bool BadgeIsEquipped(int id)
public static bool BadgeIsEquipped(int id, int playerid)
```
Returns true if there's at least 1 medal in `badges` whose medal id is `id` and is equipped to the party or to the player party member with a `trueid` of `playerid`. If `playerid` is -1, it includes every equipped medals in the search.

```cs
public static int BadgeHowManyEquipped(int id)
public static int BadgeHowManyEquipped(int id, int playerid)
```
Returns the amount of medals in `badges` whose medal id is `id` and is equipped to the party or to the player party member with a `trueid` of `playerid`.

```cs
public static int[] GetEquippedBadgeIDs()
```
Returns an array containing the medal ids in `badges` that are equipped on any player party member or the party. NOTE: If the array is empty, but the game isn't in a `pausemenu`, an array of element being -1 is returned instead of an empty array.

```cs
private static int[] GetBadgeIDs()
```
This is UNUSED, but remains functional. Returns an array containing the medal ids in `badges`. NOTE: If the array is empty, but the game isn't in a `pausemenu`, an array of element being -1 is returned instead of an empty array.

```cs
public static int HasBadge(int badgeid)
```
UNUSED: No one calls this, but it remains functional.

Returns the amount of times the medal whose id is `badgeid` is equipped to anyone. More precisely, it checks the amount of `badges` elements whose medal id is `badgeid`.

This is presumably a previous implementation of BadgeIsEquipped.

## Medals shops
Some medals can only be obtained by purchasing them in a medal shop. There are only 2 defined in the game: Merab (id 0) and Shades (id 1). The actual shops themselves are defined by a [shop system](../Entities/NPCControl/Shop%20system.md) which is also where the information of whose shop it is can be found. The informations related to the shops are also saved to the [save file](../External%20data%20format/Save%20File.md#line-4-array-line-medals-available-in-medal-shops):

- `badgeshops[X]` where `X` is the shop id: The list of medals ids currently in stock in the shop
- `avaliablebadgepool[X]` where `X` is the shop id: The ordered list of medals ids displayed (only the first ones will be displayed, the rest won't be)

Under normal circumstances, both of the lists for a given shop id should contain the same elements, but the order may differ. `avaliablebadgepool` gets shuffled during gameplay

Each `badgeshops` and `avaliablebadgepool` have a starting value that is enforced on boot by the following method:

```cs
public void SetUpBadges()
```
It is as part of [SetVariables](../MainManager/Boot%20and%20reset%20process.md#setvariables).

It sets `badgeshops[0]` (Merab) and `badgeshops[1]` (Shades) to their starting list and also set their matching `avaliablebadgepool` to be new lists containing the same elements. Here are the starting list of medals set:

- Merab: `HPPlus`, `TPPlus`, `PoisonResistance`, `SleepyResistance`, `DoublePainReal`, `Ambush`, `Pip`, `LastWind`, `ItemRecycle`, `BadDream`
- Shades: `SuperBlock`, `PoisonAttacker`, `PoisonDefender`, `StatusBoost`, `EXPBoost`

These are only truly valid when starting a new game because if a save file is loaded, both `badgeshops` and `avaliablebadgepool` gets set to the loaded values from the save file.

## Prize medals
Some medals can only be obtained during some events, typically after beating a boss fight. The way they are obtained depends on whether or not the `DoublePain` medal was equipped when the medal becomes obtainable and it involves a complex system involving [flagvars](../Flags%20arrays/flagvar.md). It's also a system used to make a medal obtainable when progressing through the game would have made the medal unobtainable otherwise.

This is what is called a prize medal. Prize medals are composed of the following:

- The medal id associated with the prize medal
- A flagvar slot reserved to hold the state of the prize medal. Here are what the value of such a flagvar slot indicates about the prize medal:
    - 0: The prize medal isn't obtainable yet, but could become obtainable later
    - 1: The prize medal is claimable from Artis
    - 2: The prize medal is available for purchase at the [Caravan](../Entities/NPCControl/Interaction/CaravanBadge.md) shop
    - 3: The prize medal was obtained and won't appear again
- An [enemy](../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md) id whose name would be displayed when claiming the prize medal from Artis

Prize medals also have their own id, but the data about them doesn't come from any TextAsset data. All the information above are hardcoded in various array fields initialised during SetVariables. This includes the amount: there are 23 prize medals defined in the game.

Here are the fields involved (all of them are arrays indexed by prize medal id):

- `prizeids`: The list of medals id associated with each prize medals
- `prizeflags`: The list of flagvar slots bound to each prize medal
- `prizeenemyids`: The list of enemy ids associated with each prize medals

And here are their hardcoded data:

|Id|`prizeids[id]`|`prizeflags[id]`|`prizeenemyids[id]`|
|-:|-------------|----------------|------------------|
|0|`SpeedUp` (Quick flea)|13|2|
|1|`WeakStomach`|17|`CordycepsAnt`<sup>1</sup>|
|2|`SpikeBod`|18|`VenusBoss`|
|3|`Depower1` (Break)|19|`Zasp`|
|4|`TPPlus`|20|`Mothiva1`|
|5|`LifeSteal`|21|`Scarlet`|
|6|`Empower`|25|`BeeBoss`|
|7|`HardCharge`|33|`Scorpion`|
|8|`Emfeeble`|34|`ZombieRoach`|
|9|`ReversePoison`|36|`Centipede`|
|10|`ResistAll`|44|`PrimalWeevil`|
|11|`TauntPlus` (Deep Taunt)|45|`Ahoneynation`|
|12|`HappyTP` (TP Core)|46|`MotherChomper`|
|13|`HeavyThrow`|48|`Kali`|
|14|`RandomStart`|51|`Carmina`|
|15|`Reflection`|52|`CordycepsAnt`<sup>1</sup>|
|16|`HPPlus`|57|-1 (see note below<sup>2</sup>)|
|17|`AntlionJaws`|61|`WaspGeneral`|
|18|`MiracleMatter`|63|`MidgeBroodmother`|
|19|`BerryFinder`|30|`Fisherman`|
|20|`NeedlePincer` (Block Heal)|31|`Zommoth`|
|21|`HealingBuzz` (Triumph Buzz)|64|`BanditLeader`|
|22|`MiracleMatter`|65|`WaspKing`|

1: In practice, these enemy ids are dummy values and never used under normal gameplay because they are never claimable from Artis. In fact, these 2 are the only ones in the game that are made obtainable by directly setting their bound flagvar to 2 (obtainable at the Caravan shop) without involving AddPrizeMedal. This means it's not possible to see the enemy name for these prize medals as they can only be obtained at the Caravan, but because it's not possible to specify "no enemy", 0 is used as a dummy enemy id value.

2: -1 is a special value reserved for beating the fight in event 175 composed of `Cenn` and `Pisci`. It indicates to not pull any enemy names when claiming the prize medal through Artis, but to instead specifically use the map dialogue line 69 (`duo of fake explorers` in English). This logic is hardcoded in event 33 (taling to Artis)

A prize medal can only be made obtainable by calling a method: AddPrizeMedal. There are 2 exceptions to this mentioned above, but every other prize medal normally goes through this method.

```cs
public static void AddPrizeMedal(int id)
```
It makes the prize medal id `id` available to be obtained or purchased. More precisely, the following is done:

- If `DoublePain` medal is equipped or flag 614 is true (HARDEST is enabled):
    - [flagvar](../Flags%20arrays/flagvar.md) of `prizeflags[id]` is set to 1 (obtainable by talking to Artis)
    - flag 56 is set to true (puts a ! emoticon above Artis)
- Otherwise:
    - flagvar of `prizeflags[id]` is set to 2 (obtainable by purchasing the medal at a Caravan shop)
- flagvar 55 is incremented (amount of prize medals avaible or obtained)

This is only manually called by the game during events and the [addprize](../SetText/Individual%20commands/Addprize.md) SetText command. The enemy id bound to each prize medal is only for presentation purposes when claiming the prize medal through Artis, but there are no automated managed of making prize medals obtainable.

Alongside this method, there's a few utility methods to work with prize medals. Here there are.

```cs
public static int[] PrizeBadges(bool caravan)
```
If `caravan` is false, returns an array of all the `prizeflags` (prize medals's bound [flagvar](../Flags%20arrays/flagvar.md) slot) elements whose value is 1 (claimable from Artis).

If `caravan` is true, returns an array of all the `prizeids` elements (prize medals's medal id) whose matching `prizeflags` (bound flagvar slot) is 2 (available for purchase at the [Caravan](../Entities/NPCControl/Interaction/CaravanBadge.md) shop).

```cs
public static int GetEnemyPrizeID(int flagvalue)
```
Returns the prize medal id whose `prizeflags` element (bound flagvar slot) is `flagvalue`.

## Secret menu codes
There are 2 secret menu codes that heavily involves medals: MYSTERY? and RUIGEE.

### MYSTERY?
This code changes the way medals are obtained throughout the game by shuffling the order they are given. Normally, when collecting one via an NPCControl's [CheckItem](../Entities/NPCControl/ObjectTypes/Item.md#checkitem) call or a [giveitem](../SetText/Individual%20commands/Giveitem.md) SetText command, the associated medal is simply given to the player.

MYSTERY? changes this so that the medal id is the return of a special method called GetRandomMedal.

```cs
public static int GetRandomMedal()
public static int GetRandomMedal(bool dontremove, bool random)
```
This method is at the center of the MYSTERY? system. It involves a reserved [flagstring](../Flags%20arrays/flagstring.md) slot for this purpose (slot 13) that contains a `,` separated list of the medals ids that have yet to be obtained and that should also be included in the shuffle (some aren't present on purpose to make them deterministic still). NOTE: If flagstring 13 is empty when calling this method, it is set to `0,1,2,3,4,5,6,7,8,9` which is a dummy value. To avoid this, the flagstring needs to be initialised before using this method.

The order of the medals ids in flagstring 13 is what matters here: it dictates the order in which the medals will be returned each time this method is called where `dontremove` and `random` are both false. Calling the method in sucession in such a fasion effectively enumerates the medal ids in the flagstrings in the same order.

If `random` is true, the medal ids will be shuffled before a random one is obtained from a random element in the list. This new order will also be reflected to the flagstring so it is a permanent shuffle that will be reflected on the next times the method is called. This is only used when creating a new save file with the code or when upgrading a save file to 1.2.0 since this version added new medals ids in the shuffled pool so it requires a new shuffle to preserve randomness.

If `dontremove` is true, the medal id obtained from the list (whether or not `random` was true or not) will not be removed from the flagstring after obtained the medal id. Having both this parameter and `random` to true still means the flagstring will change to reflect the new order, but the medal id won't be removed from it. This is used in the same way than the `random` parameter.

To summarise the way these parameters work:

- To initialise the [medal](../Enums%20and%20IDs/Medal.md) ids pool, all elements that should be in the pool should be built and the flagstring 13 set to a `,` separated list of them. Since they aren't shuffled, this medal should be called with both `dontremove` and `random` being true which won't remove anything from the pool, but it will permanently shuffle it
- To obtain a medal id from an already initialised pool, this method should be call with both `random` and `dontremove` set to false. This will obtain the first medal id, remove it from the pool and return it, but it won't shuffle the list

The parameterless overloads has both `dontremove` and `random` parameters default to a false value.

Any other combination of `random` and `dontremove` parameters aren't used by the game, but remains functional, just not recommended.

As for the MYSTERY? code itself, it also changes some specific behaviors and rendering to obfuscate the actual medal's name since this won't be known until the moment GetRandomMedal is called.

### RUIGEE
RUIGEE is a code that has big implications on medals. Here are all the medals specific changes:

- All MP cost of every medals in the PauseMenu are clamped from 0 to 1 instead of from to 0 99. This is reflected when checking for allowing the equip of all medals. It's also reflected in the medals listtype and the equipped medals listtype
- In event 197 (talking to Paton) when removing all `statbonus`, all the ones with type `MP` are excluded from being removed and they will not yield `MPPotion` items.
- In event 8 (creating a new file), the starting `maxbp` is 4 instead of the regular 5 enforced by SetVariables

## Other utility methods
The following methods are other potentially useful medals utilities.

```cs
public static string GetBadgeName(int id)
```
Returns a formatted string using `menutext[268]` to indicate a medal's name whose id is `id` followed by the language specific of the "medal" term.

The way this works is `menutext[268]` only contains the letters `i` and `m`. The `m` part gets replaced by `menutext[159]` which is the language specific "medal" term. The `i` part gets replaced by `badgedata[id, 0]` which is the medal's name whose id is `id`. 

```cs
public static void RefreshBadgeOrder()
```
Reorder `badges` by creating a new list containing the same elements, but ordered in such a way that each medal id appears in the same order as `badgeorder` medal ids.
