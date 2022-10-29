* If the [listtype](../listtype.md) isn't a questboard list
  * **Determine the text to render and if we are going to render at all via SetText according to the [listtype](../listtype.md)**
  * Create a white 9Box at the bottom left portion of the screen that will have the description
  * If `sell`:
    * Set the HUD to show the berry count.
    * Set flagvar 10 to half the buying price rounded down of the item obtained from itemdata according to the [listtype](../listtype.md) as the item type and `option` as the item id clamped from 1 to 999.
    * Prepend the input string prepared earlier with the secondary string obtained earlier followed by `-` + the sell price display line with a [Currency](../../SetText/Commands/Individual%20commands/Currency.md) command at MenuText line id 49 + `|line|`
  * Calls SetText in non [Dialogue mode](../../SetText/Dialogue%20mode.md) with the following:
    * `|single||singlebreak,` + itemdescbreak (This is 10.5 normally) + `|` + the final text determined earlier
    * `BubblegumSans` font
    * no `linebreak`
    * no `tridimensional`
    * position at (-5.25, 0.6)
    * no `cameraoffset`
    * size at (0.75, 0.75)
    * parent is the 9Box created earlier
    * no `caller`
  * Set `listdescbox` as the 9Box created earlier.
* If the currently selected quest isn't None:
  * Create the `listdescbox`.
  * Render an image of the quest id `option`'s author using librarysprites from the index obtained in boardquestdata which will have a name of `Image` and a tag of `Text`.
  * Calls SetText with the following in non [Dialogue mode](../../SetText/Dialogue%20mode.md):
    * `text`: `|size,0.75||sort,1|` + The `By:` from MenuText line id 104 + ` ` + The author of the quest id `option` obtained from boardquestdata + `|line||halfline|` + The `Difficulty:` from MenuText line id 105 + ` |stars,` + The amount of filled in stars of the quest id `option` obtained from boardquestdata + `|`.
    * `position`: (12, 0.35).
    * `parent`: `listdescbox`.
