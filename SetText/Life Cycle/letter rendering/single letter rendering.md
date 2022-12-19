* if the single letter slot is null
  - Set the single letter slot to the first free slot and if no slot is available, yield for a frame until one is free.
  - Reserve the single letter slot slot to be empty string, with a font of fonttt, with the parent being the [textholder](../../Notable%20local%20variable/textholder.md) at Vector3.Zero, using the current text [Color](../../Commands/Individual%20commands/Color.md), a sort of [Sort](../../Commands/Individual%20commands/Sort.md) and a size of [size](../../Commands/Individual%20commands/size.md).x * langOffset, [size](../../Commands/Individual%20commands/size.md).y, 1f) * 0.07, 0.0
  * Set `currentoffset` via GetLetterOffset of the single letter slot and a size of [size](../../Commands/Individual%20commands/size.md) * langoffset
  * Append the current character to the single letter slot.
  * Set the localPosition of the single letter slot to `centralize` ? (0.0 - GetTextLenght(ndd.text, fonttt) / 2.0) : 0.0, `currentline`) * langOffset + (Vector3)MainManager.letteroffset
  * If `Dropshadow` is not null
    * if the single letter slot is null
      * Sets `ds` to the first free letter slot (NOTE: NO CHECK IF A SLOT IS AVAILABLE!)
      * Reserve the `ds` slot to be empty string, with a font of fonttt, with the parent being the single letter slot at `Dropshadow`, using a half transparent black color, a sort of [Sort](../../Commands/Individual%20commands/Sort.md) - 1 and a size of Vector3.One
    * Sets `ds`.text to the single letter slot's text
