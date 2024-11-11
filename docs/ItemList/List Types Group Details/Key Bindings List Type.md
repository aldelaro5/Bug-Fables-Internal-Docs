# Key Bindings list type

Display the list of inputs of the game and their bound keyboard key.

## Options generation

`listvar` is all the integers from 0 to the amount of inputs of the game - 1.

`listredirect` is overridden to null.

## Option's [SetText](../../SetText/SetText.md) input string

The text is all these strings appended together in order:
- `|`[button](../../SetText/Individual%20commands/Button.md)
- The option 
- `,0|` 
- If [languageid](../../SetText/languageid.md) is 6 (`Russian`) and the option is between 4 and 7 inclusive (confirm, cancel, switch or toggle hud inputs), `|`[sizemulti](../../SetText/Individual%20commands/Sizemulti.md)`,0.7,1|` is appended here, otherwise, nothing is appended
- MenuText line 88 + the option

In practice, it means that it will show the currently bound keyboard key of the input followed by the input description as they all fall between line 88 and 98 of MenuText.

## Description box rendering

It uses the default rendering scheme described in [Description box rendering](../ShowItemList%20Life%20Cycle/Description%20box%20rendering.md) where the text is empty. It should be noted that under normal gameplay, this list type is not called with `showdescription` set to true.

## Input handling

The Input handling of the list is done by PauseMenu's Update instead of MainManager's.
