# Items
These methods operates on [items](../../Enums%20and%20IDs/Items.md).

```cs
public static int MixIngredients(int item1, int item2)
```
Search `recipedata` for a compatible recipe where `item1` and `item2` matches the ingredients and returns the result of the matching recipe if one is found or 8 (`Mistake`) if no such recipe exists. To search of a single item recipe, `item2` needs to be -1. The method supports having `item1` and `item2` being reversed compared to the recipe for a double item recipe since it will normalise the order before searching.

If `item1` or `item2` is 156 (`Guarana`), no search is performed and instead, the result is a random result determined among the following:

- If `item2` isn't 156 (`Guarana`):
    - 1/13 to be `RoastBerry`
    - 2/13 to be `BigMistake`
    - 1/13 to be `BerryJuice`
    - 2/13 to be `BerrySmoothie`
    - 1/13 to be `Mistake`
    - 3/13 to be `CookedJellyBean`
    - 1/13 to be `TangyJam`
    - 1/13 to be `SpicyBomb`
    - 1/13 to be `BurlyBomb`
- Otherwise (`item1` is 156, `Guarana`):
    - 2/11 to be `RoastBerry`
    - 4/11 to be `Mistake`
    - 2/11 to be `BigMistake`
    - 1/11 to be `BerryJuice`
    - 1/11 to be `CookedJellyBean`
    - 1/11 to be `BerrySmoothie`

```cs
public static string GetItemString(bool emptyinv)
```
Returns a string that represents a `,` separated list of all item id in `items[0]` (standard items inventory). If `emptyinv` is true, `items[0]` is reset to a new empty list before returning the string which empty empties the entire standard items inventory.

```cs
public static void InventoryFromString(string inp, int itemtype, bool addd)
```
Replace `items[itemtype]` with the list of item ids obtained by splitting `inp` by `,`. If `addd` is true, the items will be added to the existing list instead.

NOTE: `inp` must be a a `,` separated list of valid item ids and `itemtype` must be 0, 1 or 2 or an exception will be thrown.

```cs
public static Sprite GetItemSprite(bool badge, int id)
```
Returns `itemsprites[0, id]` if `badge` is false or `itemsprites[1, id]` if `badge` is true.

```cs
public static int HowManyItem(int type, int id)
```
Returns the number of occurences of the item whose id is `id` in `items[type]`.

```cs
private static void GetRewardTokens(int score)
```
Changes the value of flagvar 1 (amount of game tokens) depending on `score`, `battleresult` (which indicate if the minigame was beaten) and flagvar 6 (indicates the minigame played, 0 is FlappyBee and 1 is MazeGame). The value is always clamped from 2 to 999999 at the end.

More specifically, this is the logic used to determine the value (before the clamping):

- If `battleresult` is false (the minigame was failed), the value is `score` / 200.0 floored
- Otherwise, if flagvar 6 is 0 (minigame was FlappyBee), the value is `score` / 95.0 floored
- Otherwise (minigame is assumed to be MazeGame), the value is `score` / 140.0 floored

```cs
public static ItemUse GetItemUse(int id)
public static ItemUse GetItemUse(int id, int itemtype)
```
Returns the ItemUse of an item with id `id`. Check the ItemUse documentation for more details. NOTE: `itemtype` must be 0 or an exception will be thrown.

The overload without an `itemtype` parameter has its value default to 0.

```cs
public static int DoItemEffect(ItemUsage type, int value, int? characterid)
```
Process an ItemUsage with value `value` on `playerdata[characterid]` if `characterid` isn't null. Check the DoItemEffect documentation to learn more.

## Unused or broken
These methods are unused or are not functional.

```cs
public static int? GetItemHeal(int id)
```
This is UNUSED. it involved the unused BattleControl.TryHealEnemyItem coroutine.
