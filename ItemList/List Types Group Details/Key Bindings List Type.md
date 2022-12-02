# Key Bindings list type

Display the list of inputs of the game and their bound keyboard key.

## Options generation

`listvar` is all the integers from 0 to the amount of inputs of the game - 1.

`listredirect` is overridden to null.

## Option's SetText input string

The text is |[button](../../SetText/Commands/Individual%20commands/Button.md), followed by the option followed by `,0|` followed by MenuText line 88 + the option. In practice, it means that it will show the currently bound keyboard key of the input followed by the input description as they all fall between line 88 and 98 of MenuText.

## Description box rendering

It uses the default rendering scheme described in [Description box rendering](../ShowItemList%20Life%20Cycle/Description%20box%20rendering.md) where the text is empty. It should be noted that under normal gameplay, this list type is not called with `showdescription` set to true.

## Input handling

The Input handling of the list is done by PauseMenu's Update instead of MainManager's.
