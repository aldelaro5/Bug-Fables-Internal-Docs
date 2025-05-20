
## Medals

```cs
public static void AddPrizeMedal(int id)
```
Make the prize medal id `id` available to be obtained or purchased. More precisely, the following is done:

- If `DoublePain` medal is equipped or flag 614 is true (HARDEST is enabled):
    - flagvar of `prizeflags[id]` is set to 1 (obtainable by talking to Artis)
    - flag 56 is set to true (puts a ! mark above Artis)
- Otherwise:
    - flagvar of `prizeflags[id]` is set to 2 (obtainable by purchasing the medal at a Caravan shop)
- flagvar 55 is incremented (amount of prize medals avaible or obtained)

```cs
public static int HasBadge(int badgeid)
```
UNUSED: No one calls this, but it remains functional.

Returns the amount of times the medal whose id is `badgeid` is equipped to anyone. More precisely, it checks the amount of `badges` elements whose medal id is `badgeid`.

This is presumably a previous implementation of BadgeIsEquipped.

```cs
public void SetUpBadges()
```
A part of SetVariables.

Set `badgeshops[0]` (Merab) and `badgeshops[1]` (Shades) to their starting list and also set their matching `avaliablebadgepool` to be new lists containing the same elements. Here are the starting list of medals set:

- Merab: `HPPlus`, `TPPlus`, `PoisonResistance`, `SleepyResistance`, `DoublePainReal`, `Ambush`, `Pip`, `LastWind`, `ItemRecycle`, `BadDream`
- Shades: `SuperBlock`, `PoisonAttacker`, `PoisonDefender`, `StatusBoost`, `EXPBoost`

```cs
public static int GetRandomMedal()
public static int GetRandomMedal(bool dontremove, bool random)
```
TODO: This seems simple, but annoying to parse, come back to this later to explain how this works

```cs
public static string GetBadgeName(int id)
```
Returns a formatted string using `menutext[268]` to indicate a medal's name whose id is `id` followed by the language specific of the "medal" term.

The way this works is `menutext[268]` only contains the letters `i` and `m`. The `m` part gets replaced by `menutext[159]` which is the language specific "medal" term. The `i` part gets replaced by `badgedata[id, 0]` which is the medal's name whose id is `id`. 

```cs
public static void RefreshBadgeOrder()
```
Reorder `badges` by creating a new list containing the same elements, but ordered in such a way that each medal id appears in the same order as `badgeorder` medal ids.

```cs
private static int[] GetBadgeIDs()
```
This is UNUSED, but remains functional. Returns an array containing the medal ids in `badges`. NOTE: If the array is empty, but the game isn't in a `pausemenu`, an array of element being -1 is returned instead of an empty array.

```cs
public static int[] GetEquippedBadgeIDs()
```
Returns an array containing the medal ids in `badges` that are equipped on any player party member or the party. NOTE: If the array is empty, but the game isn't in a `pausemenu`, an array of element being -1 is returned instead of an empty array.

```cs
public static int BadgeHowManyEquipped(int id)
public static int BadgeHowManyEquipped(int id, int playerid)
```
Returns the amount of medals in `badges` whose medal id is `id` and is equipped to the party or to the player party member with a `trueid` of `playerid`.

```cs
public static bool BadgeIsEquipped(int id)
public static bool BadgeIsEquipped(int id, int playerid)
```
Returns true if there's at least 1 medal in `badges` whose medal id is `id` and is equipped to the party or to the player party member with a `trueid` of `playerid`. If `playerid` is -1, it includes every equipped medals in the search.

```cs
public static void AddBadge(int id)
```
Add a `badges` element where the medal id is `id` and the equip state is -2 (unequipped).

```cs
public static int[] PrizeBadges(bool caravan)
```
TODO: document the prize medals stuff separately so it's clearer how this is organised

```cs
public static int GetEnemyPrizeID(int flagvalue)
```
TODO: document the prize medals stuff separately so it's clearer how this is organised

