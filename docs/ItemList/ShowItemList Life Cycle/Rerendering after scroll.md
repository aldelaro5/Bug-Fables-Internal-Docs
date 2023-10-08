* Sets `listy` to `listlow` to indicate the list has been scrolled.
* If this is a new [ItemList](../ItemList.md), Initializes the static fields: `listmax` to `listammount`, `listlow` to 0, option to 0, `listcursor` to 0, `inlist` to true and create the cursor.
* If this is a refresh, free all the letter slots used in [ItemList](../ItemList.md) and itself.
* Resets [ItemList](../ItemList.md) to a new GameObject, set its parent to the GUICamera or to the `questboardobj` if it isn't null and set its localPosition to (`position`.x, `position`.y, 10.0).
* If [listtype](../listtype.md) is the language selection list, Call [SetText](../../SetText/SetText.md) in non [Dialogue mode](../../SetText/Dialogue%20mode.md) where the text is set to the language selection prompt using the index of option of `listvar` prepended with |[color](../../SetText/Individual%20commands/Color.md),4||[center](../../SetText/Individual%20commands/Center.md)\||[sort](../../SetText/Individual%20commands/Sort.md),20|. This effectively changes the line to match the selected language.
* If `listlow` is above 0 and `maxoptions` is higher than `listammount` (which means that there are more items than visible at once and we have scrolled below the top):
  * Setup the up arrow GUI sprite to appear at the top of the list as child of ItemList. The exact setup depends on [listtype](../listtype.md) and if we are in the pause menu.
* If we had a list in `overridedlist`, restore it and clear it.
* If we allow multi select `listsell` is true OR [listtype](../listtype.md) is storage items OR ([listtype](../listtype.md) is standard items and [flags](../../Flags%20arrays/flags.md) 349 is on), then setup the UI to instruct how to operate multiple selection.
* For each index in the visible part of the ItemList that does not exceed `maxoptions` (this means an empty ItemList will have no iteration):
  * Setup the SpriteRenderer that will contain the option as child of the ItemList. A 75% opaque rectangle will be rendered as background if we aren't paused and the [listtype](../listtype.md) isn't one about quests, settings, language list or input lists (color is green if it is in `multiselect` or white otherwise). The layer is set to 5, scaled by 1.15 in x when not paused and the position is relative to its index.
  * Set the default rendering param to send to [SetText](../../SetText/SetText.md) that can be overridden by [listtype](../listtype.md): x is -2, y is -0.15 and size is (0.75, 0.75).
  * **Obtain the text of the option according to [listtype](../listtype.md)**.
  * Calls [SetText](../../SetText/SetText.md) in non [Dialogue mode](../../SetText/Dialogue%20mode.md) using the `BubblegumSans` font in non tridimensional at the position and size determined earlier without a Camoffset with the SpriteRenderer as the parent and without a caller with the text prepended with:
    * \|[size](../../SetText/Individual%20commands/size.md),0.6,0.8| when unpaused 
    * Nothing if paused and [listtype](../listtype.md) is not the current medals list 
    * `  ` otherwise.
* If the [ItemList](../ItemList.md) isn't scrolled such that the bottom is visible:
  * Setup the down arrow GUI sprite to appear at the bottom of the ItemList as child of the `ItemList`. The exact setup depends on [listtype](../listtype.md) and if we are in the pause menu.
