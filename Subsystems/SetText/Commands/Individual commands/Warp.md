# Warp

Setup a map transfer to either a specific map id or one that is contained in a flagvar with the optional ability to set where to move the party after the new map is loaded.

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

### `map`: int | string

The map to go to. The int form indicates the map id directly and it must be a non negative int or an exception will be thrown during the transfer. 

The string form indicates to get the map id from a flagvar slot and it must have a value such that it contains `var` and a valid flagvar slot. If after removal of `var`, the result is not a valid flagvar slot, an exception will be thrown. The flagvar must contain a non negative int or an exception will be thrown during the transfer.

In either form, if the map id resolves to a value higher than the max map id, it defaults to the map NearSnakemouth.

### `x`: float

The x position to move the party after the new map is loaded. This must be a valid float or an exception will be thrown.

### `y`: float

The y position to move the party after the new map is loaded. This must be a valid float or an exception will be thrown.

### `z`: float

The z position to move the party after the new map is loaded. This must be a valid float or an exception will be thrown.

## Remarks

The map transfer won't occur during this commands's processing, but rather during [life cycle > Cleanup](../../life%20cycle.md#cleanup) at end of the SetText call. This only setup the transfer to happen at that moment.

This commands also sets [End](End.md) to true.
