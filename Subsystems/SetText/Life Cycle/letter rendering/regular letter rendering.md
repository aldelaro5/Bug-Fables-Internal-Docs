* Increment `written`
  * In [Dialogue mode](../../Dialogue%20mode.md) mode, sets [tailtarget](../../Notable%20local%20variable/tailtarget.md) to be talking
  * If in [Dialogue mode](../../Dialogue%20mode.md) mode and the current character isn't the last one, play the current bleep sound with the current pitch and volume settings
  * Get the first free letter slot and store it in letter
  * If a slot was available
    * if `ui3d`, letter's layer is set to 15 else if `tridimensional` is false, letter's layer is set to 5, 0 otherwise
    * Set the letter's tag to `Letter`
    * Reserve the letter slot to be the current char, with a font of fonttt, with the parent being the [textholder](../../Notable%20local%20variable/textholder.md) at `currentoffset`, `currentline` - 0.1, 0.0 + letteroffset, using the current text [Color](../../Commands/Individual%20commands/Color.md), a sort of [Sort](../../Commands/Individual%20commands/Sort.md) and a size of `Size`.x, `Size`.y, 1.0) * 0.07, 0.0
    * If `Dropshadow` is not null
      * Sets `ds` to the first free slot of `letterpool` (NOTE: NO CHECK IF A SLOT IS AVAILABLE!)
      * Sets `ds`'s layer to the letter's layer
      * Reserve the `ds` slot to be letter's text, with a font of fonttt, with the parent being [textholder](../../Notable%20local%20variable/textholder.md) at the letter's localPosition + `Dropshadow`, using a half transparent black color if `Fadeletter` is false and clear color otherwise, a sort of the current [Sort](../../Commands/Individual%20commands/Sort.md) and a size of the letter's scale.
      * If `Fadeletter` is true, starts a fade of `ds`'s MeshRender to half transparent over the course of 200 frames
      * Increment the letter's MeshRenderer's sortingOrder
    * If any of the font effects is enabled, add a [FontEffects](../../Related%20Systems/FontEffects.md) to letter with the current effects desired (also sets `superglitch`)
    * Add GetLetterOffset of the current character with font fonttt and `Size`.x to `currentoffset`
  * Sets `maxlength` to `currentoffset` if it is larger
  * Sets `textwidth` to `maxlength`
  * Sets `lasttextcenter` to `position`.x - `maxlenght` / 2.0
  * If [Center](../../Commands/Individual%20commands/Center.md) is enabled, sets the [textholder](../../Notable%20local%20variable/textholder.md)'s localPosition to `position`.x - `maxlenght` / 2f, `position`.y, `position`.z
  * if (`Minibubble` || (we are in [Dialogue mode](../../Dialogue%20mode.md) mode && (([Speed](../../Commands/Individual%20commands/Speed.md) > 0.0 && !instance.`skiptext`) || [Noskip](../../Commands/Individual%20commands/Noskip.md)))), yield for [Speed](../../Commands/Individual%20commands/Speed.md) seconds
  * `if (((dialogue && speed > 0f && !instance.skiptext) || minibubble) && char.IsPunctuation(text[i]) && i + 1 < text.Length - 1 && text[i + 1] != '|' && text[i + 1] != ')' && text[i + 1] != '¿' && text[i + 1] != '¡' && text[i] != '\'' && text[i] != '/' && text[i] != '¿' && text[i] != ')' && text[i] != '¡' && (!char.IsPunctuation(text[i + 1]) || text[i + 1] != '.' || text[i + 1] != '!' || text[i + 1] != '?' || text[i + 1] != ')' || text[i + 1] != '？' || text[i + 1] != '、' || text[i + 1] != '。' || text[i + 1] != '！' || text[i + 1] != '¿' || text[i + 1] != '¡'))`, yield for 0.15 seconds
