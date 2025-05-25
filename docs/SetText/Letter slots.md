# Letter slots
SetText operates on specialised TextMesh that are configured to be rendered. They are pooled in an array field called `letterpool` and the game allocates 500 of them on boot.

Such a TextMesh is refered to as a letter slot. Letter slots are the building blocks of SetText's rendering. Every piece of text rendered from SetText is done with letter slots. Since the game enforces a limit of 500, it means that at any given moment, at most 500 letter slots may be in use by SetText.

What a letter slot represents depends on the rendering system. In [regular letter rendering](Letter%20Rendering%20Methods/Regular%20Letter%20Rendering.md), each letter slot is a single character while in [single letter rendering](Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md), each letter slot is a complete line. The main purpose of the latter is to use the least amount of letter slots whily having much less, but sufficient compatibility with the various visual effects the former rendering method provides.

Since 500 characters isn't a whole lot and it would be easy to run out of letter slots, SetText makes sure to periodically free the letter slots it no longer needs to render. Doing this allows to use the same letter slots that were used previously after they were freed. So effectively, the 500 limit only applies for a single moment: there cannot be more than 500 letter slots rendered on screen at the same time, but reusing letter slots that were freed is fine.

## Allocating a letter slot
Allocating a letter slot involves the method NewLetter:

```cs
private static TextMesh NewLetter()
private static TextMesh NewLetter(string id)
```
It creates a GameObject whose name is `letterX` where `X` is the value of `id` with a TextMesh that is setup as a SetText letter slot and is then returned. The parameterless version of the method is UNUSED, but remains functional and it calls the other overload with an `id` value of empty string.

This is primarily called as part of LoadEssentials when setting up the initial 500 letter slots in the `letterpool`, but it's also possible GetEmptyLetter (more details in a section below) calls it if it finds out that a letter slot is null.

A letter slot is a TextMesh that is setup with the following characteristics which are optimised for SetText usage:

- Childed to the MainCam (where MainManager resides) with a local z position of 10.0 on layer 5 (`UI`)
- richText is disabled
- fontStyle of Normal
- anchor set to LowerLeft
- MeshRenderer's shadowCastingMode set to Off
- MeshRenderer's allowOcclusionWhenDynamic disabled

What this means is that under normal cirumstances, all letter slots are configured like this on boot and if for some reasons, one becomes null, it will eventually get recreated by this method when obtaining a free slot.

## Obtaining a free letter slot
As for how to obtain a free slot, this is where the method GetEmptyLetter comes in:

```cs
public static TextMesh GetEmptyLetter()
```
It finds and allocate if needed a free letter slot from `letterpool` and return it or return null if no letter slots is available. If the letter slot is null, it is allocated using NewLetter with its `letterpool` index ToString as the id before being returned. For any non null letter slots, only those with an text of empty string are considered free by this method.

## Reserving a free letter slot
After obtained a free letter slot, it needs to be reserved by SetText. This is done by SetLetter:

```cs
public static void SetLetter(ref TextMesh letter, int fontid, string text, Transform parent, Vector3 pos, Color color, int sortorder, Vector3 size)
```
It configures the letter slot `letter` with the following:

- SetFont(`letter`, `fontid`) called (more details in a section below)
- Childed to `parent`
- Local position of `pos`
- text of `text`
- anchor of LowerLeft
- color and MeshRenderer's material.color of `color`
- angles of Vector3.zero
- scale of `size`
- MeshRenderer's sortingOrder of `sortorder`

Since the text isn't empty, it effectively reserves the letter slot. GetEmptyLetter will not be able to return it until it is free.

## Freeing a reserved letter slot
In order for SetText to free a letter slot, the method DisableLetter needs to be called on it:

```cs
private static void DisableLetter(TextMesh input)
```
It either destroys `input` if it doesn't have a `Letter` tag (meaning it's not an active letter slot) or disable the letter slot if it does have a `Letter` tag such that it is considered free again. The disablement process goes as follow:

- Set `input`.text to empty string
- If `input` has a FontEffects, it is destroyed
- `input` gets childed to the MainManager instance
- `input`.tag is reset to `Untagged`

It's the prefered method for SetText to disable letter slots.

It's possible to disable all letter slots childed to a GameObject using a method with the same name:

```cs
private static void DisableLetter(Transform input)
private static void DisableLetter(Transform input, bool destroy)
```
It performs the following on `input`:

- Destroys all children GameObject with a ButtonSprite component
- Call DisableLetter on all children's TextMesh
- If `destroy` is true, `input` gets destroyed

The overload without a `destroy` parameter has its value default to true.

There's an even more useful helper method to batch free everything childed to a GameObject: DestroyText.

```cs
public static void DestroyText(Transform parent)
public static void DestroyText(Transform parent, bool destroy)
```
It Calls DisableLetter on every child of `parent` with a tag of `Text` where DisableLetter has a destroy parameter of `destroy` if a value is sent (otherwise, the overload used is DisableLetter(Transform) which behaves as if true was sent).

It's mainly useful for destroying large amount of UI text that were rendered with SetText in non dialogue mode.

## Configuring a letter slot font
While SetLetter already allows to configure the font, it's possible to do it manually and to change it after the fact by calling SetFont:

```cs
public static void SetFont(TextMesh letter, int fontid)
```
It configure `letter` such that it uses the font specified by `fontid` including setting the proper material (stored in `fontmat`) and the font shader (`fontmat[0].shader`) on the MeshRenderer (all `fontmat` uses the same 3D text shader). More precisely, the font used will be `fonts[x]` where `x` is FontID(`fontid`) and it will also dictate the `fontmat` to use in the same manner

Here's the signature of FontID:

```cs
public static int FontID(int id)
```
Returns `id` mapped to a font id with the following logic:

- Any negative numbers maps to their absolute value + 1
- 2 (UNUSED) maps to 1 (`D3Streetism`)
- Any other numbers of 0 or above are unchanged

This mapping is documented in the [fonttype](Notable%20states.md) documentation.

## Calculating letter slots spacing
A big part of SetText is the ability to automatically break lines such that they render on the textbox without overflowing. Doing this requires to know the rendered space a letter slot takes on the screen in regular letter rendering.

This is what the method GetLetterOffset gives:

```cs
public static float GetLetterOffset(char letter, int fontid, float size)
```
It returns the amount of space `letter` takes horizontally when rendered by SetText in regular letter rendering using a font with id `fontid` and a size of `size`. More precisely, this is calculated the following way:

- If `letter` is ` `, the result is 0.3 * `size`
- If `letter` is not contained in FontID(`fontid`), the result is 0.3 * `size`
- If nether of the above cases applies, the result is (the CharacterFont's advance * 0.75 - 2.0) * (`size` / the fontSize of the Font)

There's also a specialised version for single letter rendering when a [backbox](Individual%20commands/Backbox.md) command is involved to calculate its position and scale:

```cs
private static float GetLetterOffset(TextMesh letter, float size)
public static float GetLetterOffset(char letter, int fontid, float size)
```
It returns (`letter`.bounds.extents.x * 2.0 - 2.0) * `size`. If `letter` is null, 0.3 * `size` is returned instead.

Even more useful for [OrganizeLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) though is the method GetTextLength:

```cs
public static float GetTextLenght(string text, int fontid)
```
It returns the sum of all GetLetterOffset values of each char in `text` using a fontid of `fontid` and a size of 1.0. Each ` ` in `text` is counted as 0.3 without calling GetLetterOffset.

## TempLetter
There's an UNUSED (and broken) coroutine related to letter slots: TempLetter.

```cs
public static IEnumerator TempLetters(string text, int[] spacebreaks, bool rainbow, Vector3 posLeave0ToCenter, float showtime)
```
This is UNUSED and left in a broken state, but it can technically be called. It is NOT recommended to call this coroutine.

Temporarily renders `text` using newly allocated letter slots for `showtime` amount of seconds with a rainbow font effect if `rainbow` is true. The `spacebreaks` and `posLeave0ToCenter` parameters don't do anything and their values should always be null and Vector3.zero respectively.

This coroutine is broken because the letter slots used aren't configured properly (wrong positioning, wrong font and wrong font sizes). This results in the text not rendering properly with the default Unity font.
