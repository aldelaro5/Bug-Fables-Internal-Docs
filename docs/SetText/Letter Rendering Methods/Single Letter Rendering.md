# Single Letter Rendering

This letter rendering method is disabled by default, but it can be opted in using [single](../Individual%20commands/Single.md) so it needs to be explicitly toggled on. It involves rendering each line as individual TextMesh objects (called letter slots) with a maximum of 500 lines that can be rendered using this method alone.

It was introduced relatively late in development to counter [Regular Letter Rendering](Regular%20Letter%20Rendering.md)'s main problem: inefficiency. Instead of taking one letter slot per letter, single letter rendering takes one letter slot per line. This significantly reduces the amount of letter slot taken which has shown to increase performances. It also allows to render more letters on screen at once.

This comes at a huge cost: compatibility. Because the entire letter slot is a whole line, not all features like font effects and other rendering [Commands](../Commands.md) work correctly as they weren't all made to support this rendering method. As a result, some features still aren't using this rendering method mostly because it uses an incompatible feature. 

Notably, lacking features are [bleep](../Individual%20commands/Bleep.md) playing, [backtracking](../Related%20Systems/Backtracking.md) accumulation, [triui](../Individual%20commands/Triui.md), `tridimensional`, any [FontEffects](../Related%20Systems/FontEffects.md), yield on letters and others [Commands](../Commands.md) involving sprite rendering does not work correctly such as [icon](../Individual%20commands/Icon.md), [button](../Individual%20commands/Button.md) and [stars](../Individual%20commands/Stars.md). 

The performance gain is however still significant to use it when it is possible such as in UI text which was why it was introduced in the first place. The kerning turns out to be very close to [Regular Letter Rendering](Regular%20Letter%20Rendering.md) which means the text appearance doesn't change much, but spaces aren't physical gaps anymore which means they visually are narrower

## Rendering process

* if the single letter slot is null
    * Set the single letter slot to the first free slot and if no slot is available, yield for a frame until one is free.
    * Reserve the single letter slot slot to be empty string, with a font of [fonttype](../Notable%20states.md#fonttype), with the parent being the [textholder](../Notable%20states.md#textholder) at Vector3.zero, using the current text [color](../Individual%20commands/Color.md), a sort of [sort](../Individual%20commands/Sort.md) and a size of (size.x, size.y, 1.0) * 0.07. NOTE: If [languageid](../languageid.md) is 6 (`Russian`), the size is (size.x * 0.9, size.y, 1.0) * 0.07 which decreases all x sizes by 10%
* Add GetLetterOffset of the single letter slot with a size of size.x to the current offset (size.x * 0.9 if `languageid` is `Russian`)
* Append the current letter to the single letter slot.
* Set the localPosition of the single letter slot to (`x`, current line, 0.0) * `y` + (0.0, -0.1) where `x` is 0.0 or (0 - the length of the letter slot's text / 2.0) if [center](../Individual%20commands/Center.md) is enabled and `y` is 1.0 unless `languageid` is `Russian` where it's 0.9
* If [dropshadow](../Individual%20commands/Dropshadow.md) is enabled
    * If the `ds` letter is null
        * Sets `ds` to the first free letter slot (NOTE: NO CHECK IF A SLOT IS AVAILABLE!)
        * Reserve the `ds` slot to be empty string, with a font of fonttype, with the parent being the single letter at the dropshadow offset, using a half transparent black color, a sort of [sort](../Individual%20commands/Sort.md) - 1 and a size of Vector3.one
    * Sets `ds`'s text to the single letter's text
