# Languages list type

Display the list of the available languages of the game.

## Options generation

`listvar` is the integers between 0 and 6, which represents the [languageid](../../SetText/languageid.md) of the game.

`listredirect` is overridden to null.

## List render on refresh

This list will always rerender whenever a refresh is done regardless if it needs to be scrolled or not. This is done to update the help text to the option being hovered on during the selection. This prompt is refreshed by doing a [SetText](../../SetText/SetText.md) call in non [Dialogue mode](../../SetText/Dialogue%20mode.md) where the input string is the text in languagehelp of the current option prepended with |[color](../../SetText/Individual%20commands/Color.md),4||[center](../../SetText/Individual%20commands/Center.md)\||[sort](../../SetText/Individual%20commands/Sort.md),20|, the position is (1.0, 4.0) and the parent is the [ItemList](../ItemList.md). This call is done on every refresh of the list.

## Option's [SetText](../../SetText/SetText.md) input string

The text is |[sort](../../SetText/Individual%20commands/Sort.md),20||[color](../../SetText/Individual%20commands/Color.md),4| followed by the name of the language of the option in languagenames.

The size of the text is overridden to Vector2.one.

## Description box rendering

It uses the default rendering scheme described in [Description box rendering](../ShowItemList%20Life%20Cycle/Description%20box%20rendering.md) where the text is empty. It should be noted that under normal gameplay, this list type is not called with `showdescription` set to true.

## Cursor rendering

Normally, any listtype where `questboardobj` is null and not handled by PauseMenu will have the cursor progressively go towards the current option using `listcursor` by an offset in the x position of 2.0 and 0.75 in the y position (which is around the right side of the screen). This list type is special because it changes this offset to be -2.0 on x and removes the y one. This is done because this is the only list like this where the options are positioned at the center of the screen rather than on the right.

## Confirmation handling

The confirmation will cause the [languageid](../../SetText/languageid.md) to be set to the selected option, a complete rewrite (or creation) of `config.dat` and SetVariables to be called which updates all the language dependent data of the game. The list is then destroyed with the [ItemList State Machine](../ItemList%20State%20Machine.md) being reset and finally, the Intro method on the StartMenu is called which starts the title screen.

Finally, if the [languageid](../../SetText/languageid.md) is 4 (`German`), some `guisprites` gets overriden to their German counterpart (these are all the gui sprites where a medal icon can be seen as the letter inside changes from an "M" to an "A"):

- `guisprites[61]` becomes `guisprites[224]`
- `guisprites[109]` becomes `guisprites[225]`
- `guisprites[75]` becomes `guisprites[226]`

## Cancellation handling

It it not possible to cancel in this list. Nothing will happen if it is attempted.
