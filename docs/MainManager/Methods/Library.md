# Library
These methods specifically operates on anything related to `librarystuff` which is a 2 dimensional array containing various flags saved on the [save file](../../External%20data%20format/Save%20File.md) that tracks the unlocking of various [discoveries](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md), [bestiary](../../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md), [recipes](../../Enums%20and%20IDs/librarystuff/Recipes%20entry.md), [records](../../Enums%20and%20IDs/librarystuff/Records%20entry.md) and [areas](../../Enums%20and%20IDs/librarystuff/Areas.md) entries.

```cs
public static bool[] GetLibraryBools(int page)
```
Returns a new array whose elements is `librarystuff[page]` meaning the 256 flags array matching the `page`.

```cs
public static bool HasRecipe(int id)
```
This is UNUSED, but remains functional. Returns the flag value matching the recipe entry in `librarystuff` where `id` is the resulting item in the recipe. If no such recipe exists, false is always returned.

```cs
public static int GetRecipeID(int id)
```
Returns the recipe id from `libraryorder` where the resulting item id is `id` or -1 if no such recipe exists.

```cs
private static int[] GetLibraryIDs(int index)
```
Get the array of effectively used `libraryorder[index]` (meaning the length is `librarylimit[index]`). Intuitively, it returns an array of all valid `librarystuff[index]` id so it acts as a Library enum value.

```cs
private static int[] GetLibraryOrder(int id)
```
Returns all effectively used (meaning length is `librarylimit[id]`) entry ids in `libraryorder` where the first dimension index is `id` (so it acts as a Library enum value).

```cs
public static int GetEnemyPortrait(int id)
```
Obtains the `librarysprites` index of the portrait sprite associated with the enemy whose id is `id`. There's some special cases logic to this which is documented in the GetEnemyPortrait documentation.

```cs
public static void UpdateJounal()
public static void UpdateJounal(Library type, int variable)
```
If `type` is Bestiary and the game is `inbattle`, `librarystuff[1, variable]` is set to true which mark the bestiary entry as obtained.

Otherwise, the following happens to mark a `librarystuff` flag as obtained if applicable as well as revealing in the HUD the obtention of a flag:

- If `variable` isn't negative, `librarystuff[type, variable]` is set to true which marks the entry as obtained
- `discoveryhud` is set to 350.0 which reveals the HUD element
- The `Disc` animation clip plays on the child of `discoverymessage` or the `Arch` clip if `type` is Logbook
- The 2nd, 3rd and 4th child of `discoverymessage` have their enablement changed depending on `type`:
    - Logbook: Only the 4th child is enabled, 2nd and 3rd are disabled
    - Other `type` values: Only the 3rd child is enabled, 2nd and 4th are disabled

The parameterless overload has the `type` value default to Discovery and `variable` to -1. This has the effect of not doing any changes to `librarystuff`, but still reveal `discoveryhud`.
