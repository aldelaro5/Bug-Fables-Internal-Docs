# Equipped [Medal](../../Enums%20and%20IDs/Medal.md)s list type

Display a list of [medal](../../Enums%20and%20IDs/Medal.md)s from `multilist`, not to be confused with [Medals List Type](Medals%20List%20Type.md). This is intended to be used to display the list of equipped medals.

## Options generation

`listvar` will set to `multilist` which is implied to have been set after the list's creation before a refresh.

The list type is overridden to [Medals List Type](Medals%20List%20Type.md).

`listredirect` is overridden to null.

## Option's [SetText](../../SetText/SetText.md) input string

The behavior is the same then the overridden [Medals List Type](Medals%20List%20Type.md).

## Description box rendering

The behavior is the same then the overridden [Medals List Type](Medals%20List%20Type.md).

## Remarks

Under normal gameplay, this list is only reachable via PauseMenu after pressing the show HUD input on the medals window. When pressing that input, the handler in PauseMenu's Update will be the one that sets `multilist` to the medals of the equipped bug. Then, UpdateText is called which is the one that calls the appropriate [ShowItemList](../ShowItemList.md) with this list type.

It behaves exactly the same than [Medals List Type](Medals%20List%20Type.md) with the only differences that it uses the [medal](../../Enums%20and%20IDs/Medal.md)s only in `multilist` instead of all the ones the player has.
