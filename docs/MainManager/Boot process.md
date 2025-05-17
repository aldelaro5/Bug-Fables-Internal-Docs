
## Boot and reset process

```cs
private static IEnumerator LoadEverything()
```
The main startup coroutine of the game, called by Start.

This is the coroutine that dispatches the main bootstrapping tasks such as setting up some fields like `events` and overall maintains consistencies between game resets (such as calling SetRenderTexture(0) to turn it off). The main work of the game's boot is done by the LoadEssentials coroutine and the SetVariables method which are called by LoadEverything. The coroutine contains multiple frame yields to allow the loading icon to somewhat (even if not consistently) animate via its attached SpriteBounce during the loading process.

```cs
private void FontSet()
```
A part of LoadEverything.

This method calls SetText in non dialogue mode in such a way that all letters contained in each fonts and whose characters are in any `letterPromptHelp` will be rendered under the game's normal viewport. This is a caching optimisation to force Unity to preload most letters and fonts materials used for future text rendering which avoids the possibility of stutters when loading any of these assets.

The SetText call is done with a position of (0.0, 0.0, -1.0), parent of `MainCamera` and a string of `|single|` followed by a list of font commands with all valid indexes each being superceded with every letters of every `letterPromptHelp`.

```cs
public static void LoadLangSpecific()
```
This method applies a German language specific adjustement where some medals UI icons gets their sprites changed such that they display an "A" instead of an "M". It is only called as part of event 22 (loading a save file) and on Update when processing a confirmation in the language selection listtype. This ensures that the adjustement is applied or not depending on the language being selected.

```cs
public static Entity_Data[] LoadEntityData()
```
A part of LoadEssentials.

Loads and return the array value that `endata` should be assigned to which contains all data belonging to each specific animid. The data comes from the `Data/EntityValues` TextAsset and the resulting array is indexed using the enum format of animids (meaning the index 0 = None).

```cs
private IEnumerator LoadEssentials()
```
A major part of LoadEverything which manages the boot process. It is expected that this coroutine is set to the `chaptername` field which can be used to track if it's still in progress (it will be set to null at the end of the coroutine).

This coroutine loads most cached and core assets of the game so they can be referenced easily without load calls later. Most of these assets are stored in fields of MainManager (most of them are static) and they aren't expected to change much after. The coroutine contains multiple frame yields to allow the loading icon to somewhat (even if not consistently) animate via its attached SpriteBounce during the loading process.

Alongside loading assets, the coroutine also does the following:

- Initialises Physics.gravity to (0.0, -40.0, 0.0)
- Calls Input.GetJoyButtons
- Calls Input.SetDefaultKeys

```cs
private static void LoadItemSprites()
```
A part of SetVariables.

This method intialises `itemsprites` as a [2, 256] array of sprites where the [0, `i`] array contains the Items sprites (each have an item id of `i`) and the [1, `m`] array contains the Medals sprites (each have a medal id of `m`). The sprites comes from the `Sprites/Items/items0` and `Sprites/Items/items1` sprites arrays.

The sprite array used and the index used in that sprite array for each items or medals are determined with a very specific logic:

- For items and where the item id is below 176, the sprite is pulled from `Sprites/Items/items0` where the sprite array index is the same as the item id
- For items and where the item id is 176 or above, the sprite is pulled from `Sprites/Items/items1` where the sprite index is 176 - the item id
- For medals and where the matching `badgedata`'s field 8 is negative, the sprite is pulled from `Sprites/Items/items0` where the sprite index is 176 + the medal id
- For medals and where the matching `badgedata`'s field 8 isn't negative, the sprite is pulled from `Sprites/Items/items1` where the sprite index is the matching `badgedata`'s field 8

```cs
private static Sprite GetMoreItem(int i, int offset = 176)
```
This is a part of LoadItemSprites that loads a sprite from `Sprites/Items/items1` using the index `offset` - `i`.

```cs
public void SetVariables()
```
A part of LoadEverything which is a part of the boot process, but it is also called when confirming a language during Update (language selection listtype) and on StartMenu's Start.

This method initialises a lot of instance and static fields of MainManager to their initial sensible values. It also is where a lot of TextAsset's data is loaded via Resources.Load calls. This is also where SetUpBadges, ChangeParty({0, 1}, true), LoadItemSprites and InputIO.LoadSettings are called.

```cs
public static void Reset()
```
Reloads the main scene. This is equivalent to resetting the entire game which happens normally when exiting to the main menu.
