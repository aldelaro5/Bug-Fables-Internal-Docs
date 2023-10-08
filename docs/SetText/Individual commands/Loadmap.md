# Loadmap

Reload the current [Map](../../Enums%20and%20IDs/Maps.md) or load another one

## Syntax

(1) (Reload the current map)

````
|loadmap|
````

(2)

````
|loadmap,map|
````

## Parameters

### `map`: int

The [Map](../../Enums%20and%20IDs/Maps.md) id to load. This must be a valid [Map](../../Enums%20and%20IDs/Maps.md) id or an exception will be thrown.

## Remarks

After the load is complete, this command will yield for a frame.
