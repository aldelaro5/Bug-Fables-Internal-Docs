# Shoppool

Add a medal to a medal shop's pool by [Medal](../../../Enums%20and%20IDs/Medal.md) id or one coming from a [flagvar](../../../Flags%20arrays/flagvar.md), remove all medals from a specific shop's pool using a store id or one coming from a [flagvar](../../../Flags%20arrays/flagvar.md) or remove all medals from all shop pools.

## Syntax

(1) (Removes all medals from all medal shops's pools)

````
|shoppool,reset|
````

(2)

````
|shoppool,reset,storeid|
````

(3)

````
|shoppool,storeid,medalid|
````

## Parameters

### `reset`

This parameters indicates to use syntax (1) or (2). Any other value will be interpreted as using syntax (3).

### `storeid`: int | `var`int

The medal shop store id or a [flagvar](../../../Flags%20arrays/flagvar.md) containing it to add a [Medal](../../../Enums%20and%20IDs/Medal.md) to or to remove all [Medal](../../../Enums%20and%20IDs/Medal.md). The resolved value must be a valid medal shop id (0 for Merab's, 1 for Shades's) or an exception will be thrown. The string form must contain `var` and have an int portion be a valid [flagvar](../../../Flags%20arrays/flagvar.md) or an exception will be thrown.

### `medalid`: int | `var`int

The [Medal](../../../Enums%20and%20IDs/Medal.md) is to add to the shop pool or a [flagvar](../../../Flags%20arrays/flagvar.md) containing it. The resolved value must be a valid [Medal](../../../Enums%20and%20IDs/Medal.md) id or an exception will be thrown. The string form must contain `var` and have an int portion be a valid [flagvar](../../../Flags%20arrays/flagvar.md) or an exception will be thrown.

## Remarks

This manages the shop pools which is saved at line 5 on the [Save File](../../../Data%20format/Save%20File.md), it does not deal with the available pools (which is line 4 on the [Save File](../../../Data%20format/Save%20File.md)). This means that it will not immediately cause changes to the displayed [Medal](../../../Enums%20and%20IDs/Medal.md)s until the available pool is updated. Removing a [Medal](../../../Enums%20and%20IDs/Medal.md) from a medal shop from both lists after purchasing it can be done with [Removebadgeshop](Removebadgeshop.md).

This command is only used for testing purposes in the TestRoom.

### A different way to add a medal to both the shop pool and the available shop pool

Interestingly, this command accidentally allows a use case where it is possible to add a [Medal](../../../Enums%20and%20IDs/Medal.md) to a shop pool, but also have the available pool take into account this change. It is done by processing this command in syntax (3) and later on, process a [Rerollshops](Rerollshops.md) which will have the secondary effect of regenerating the available pool using the updated shop pool. This allows to add a [Medal](../../../Enums%20and%20IDs/Medal.md) to shop and have it immediately available using SetText. Alternatively, it is possible to simply change [Area](Area.md) which will automatically do the same task.

This was not intended to be possible because this command was only meant as a testing command, not something used in game and [Rerollshops](Rerollshops.md) wasn't implemented in the 1.0.0 release of the game, but was added later as a way to shuffle the existing pools that would already have been updated. The game never intended to use SetText to add a [Medal](../../../Enums%20and%20IDs/Medal.md) to a shop because it uses [Event](Event.md)s to do so.
