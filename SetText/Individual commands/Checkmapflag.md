# Checkmapflag

Redirect to a [Dialogue line id](../Common%20commands%20id%20schemes/Dialogue%20line%20id.md) if a specific mapflags slot is true or continue processing as normal if it is false.

## Syntax

````
|checkmapflag,mapflags,redirect|
````

## Parameters

### `mapflag`: int

The mapflags to check its value. This must be a valid mapflags slot or an exception will be thrown. Since there are only 10 mapflags, valid values ranges from 0 to 9 inclusive.

### `redirect`: int

The [Dialogue line id](../Common%20commands%20id%20schemes/Dialogue%20line%20id.md) to redirect to if `mapflag`'s slot ends up being true. This must be a valid [Dialogue line id](../Common%20commands%20id%20schemes/Dialogue%20line%20id.md) or an exception will be thrown.

## Remarks

`map.mapflags` is an array of 10 bools that belongs to the current map, but is reset explicitly on the map's Start and unlike other flags arrays, this one is never saved on the [Save File](../../Save%20File.md). This gives it a very ephemeral and temporary usage because all their values are reset on every map loads.

Their value can be set using [mapflag](Mapflag.md).
