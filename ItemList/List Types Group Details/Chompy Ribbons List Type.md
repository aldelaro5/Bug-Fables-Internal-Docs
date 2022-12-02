# Chompy Rubbons list type

Display a list of items from `multilist`. This is intended to show the Chompy Ribbons available.

## Options generation

`listvar` will set to `multilist` which is implied to have been set after the list's creation before a refresh.

They type is overridden to the standard [Items List Type](Items%20List%20Type.md).

`listredirect` is overridden to null.

## Option's SetText input string

The behavior is the same then the overridden standard [Items List Type](Items%20List%20Type.md).

## Description box rendering

The behavior is the same then the overridden standard [Items List Type](Items%20List%20Type.md).

## Confirmation handling

Confirmation is handled by MainManager's Update. First, the [flagvar](../../Flags%20arrays/flagvar.md) of the `storeid` is set to the selected option (bug?). Then, the list gets destroyed which ends this list's processing and resets the [ItemList State Machine](../ItemList%20State%20Machine.md).

## Remarks

While the overridden type is the standard [Items List Type](Items%20List%20Type.md), it is able to display any kind of items including key items. In fact, under normal gameplay, this is used to display only key items being the ribbons Chompy has available. The entire management after confirmation is done by BattleControl's Chompy coroutine.
