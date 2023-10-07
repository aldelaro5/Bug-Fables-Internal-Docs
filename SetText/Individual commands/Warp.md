# Warp

Setup a [maps](../../Enums%20and%20IDs/Maps.md) transfer to either a specific [maps](../../Enums%20and%20IDs/Maps.md) id or one that is contained in a [flagvar](../../Flags%20arrays/flagvar.md) with the optional ability to set where to move the party after the new [maps](../../Enums%20and%20IDs/Maps.md) is loaded.

## Syntax

(1)

````
|warp,map|
````

(2)

````
|warp,map,x,y,z|
````

## Parameters

### `map`: int | `var`int

The [maps](../../Enums%20and%20IDs/Maps.md) to go to. The int form indicates the [maps](../../Enums%20and%20IDs/Maps.md) id directly and it must be a non negative int or an exception will be thrown during the transfer. 

The string form indicates to get the map id from a [flagvar](../../Flags%20arrays/flagvar.md) slot and it must have a value such that it contains `var` and a valid [flagvar](../../Flags%20arrays/flagvar.md) slot. If after removal of `var`, the result is not a valid [flagvar](../../Flags%20arrays/flagvar.md) slot, an exception will be thrown. The [flagvar](../../Flags%20arrays/flagvar.md) must contain a non negative int or an exception will be thrown during the transfer.

In either form, if the map id resolves to a value higher than the max map id, it defaults to the [map](../../Enums%20and%20IDs/Maps.md) NearSnakemouth.

### `x`: float

The x position to move the party after the new [map](../../Enums%20and%20IDs/Maps.md) is loaded. This must be a valid float or an exception will be thrown.

### `y`: float

The y position to move the party after the new [map](../../Enums%20and%20IDs/Maps.md) is loaded. This must be a valid float or an exception will be thrown.

### `z`: float

The z position to move the party after the new [map](../../Enums%20and%20IDs/Maps.md) is loaded. This must be a valid float or an exception will be thrown.

## Remarks

The [map](../../Enums%20and%20IDs/Maps.md) transfer won't occur during this commands's processing, but rather during [Cleanup](../Life%20Cycle.md#cleanup) at end of the SetText call. This only setup the transfer to happen at that moment.

This commands also sets [end](End.md) to true.
