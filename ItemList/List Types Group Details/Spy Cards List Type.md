# Spy Cards list type

Display a list of spy cards provided by `multilist`. This is intended to display the list of available Spy Cards.

## Options generation

`listvar` will set to `multilist` which is implied to have been set after the list's creation before a refresh ordered by the cards order from the game data.

## Option's [SetText](../../SetText/SetText.md) input string

The text is set to the enemy name of the enemy id associated with the card id from carddata. After, [flagvar](../../Flags%20arrays/flagvar.md) 6 is set to `option`.

The x position of the text is overridden to -2.65.

## Description box rendering

The behavior is the same then [Caravan Prize Medals List Type](Caravan%20Prize%20Medals%20List%20Type.md) (bug?).
