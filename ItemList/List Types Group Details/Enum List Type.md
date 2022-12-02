# Enum list type

Display a list of enums values for a given type, but render as if it was the corresponding list type.

## Options generation

`listvar` is all the integers from 0 to the amount of enum values for the corresponding type - 1.

Additionally, the list type is overridden to be the one that corresponds to the enum. For Items, it's standard [Items List Type](Items%20List%20Type.md) (but it will include all standard and key items values), for BadgeType, it's [Medals List Type](Medals%20List%20Type.md), for Skills, it's the first party member's [Skills List Type](Skills%20List%20Type.md) (but it will include all skills) and for [Maps](../../Enums%20and%20IDs/Maps.md), it's its own list type.

## Option's SetText input string

If the list type has not been overridden, this is assumed to be the [Maps](../../Enums%20and%20IDs/Maps.md) enum list type in which case, the text is the textual representation of the enum value and the x position of the text is overridden to -2.65.

Otherwise, the behavior is as described in the overridden list type document.

## Description box rendering

If the list type has not been overridden, this is assumed to be the [Maps](../../Enums%20and%20IDs/Maps.md) enum list type in which case, it uses the default rendering scheme described in [Description box rendering](../ShowItemList%20Life%20Cycle/Description%20box%20rendering.md) where the text is empty.

Otherwise, the behavior is as described in the overridden list type document.

## Confirmation handling

Confirmation is handled by MainManager's Update. First, the [flagvar](../../Flags%20arrays/flagvar.md) of the `storeid` is set to the selected option (bug?). Then, the list gets destroyed which ends this list's processing and resets the [ItemList State Machine](../ItemList%20State%20Machine.md).
