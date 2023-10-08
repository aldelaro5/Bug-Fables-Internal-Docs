# Mapflag

Changes a mapflags slot using a specific id directly or one coming from a [flagvar](../../Flags%20arrays/flagvar.md) slot

## Syntax

(1)

````
|mapflag,mapflags,value|
````

(2)

````
|mapflag,var,flagvar,value|
````

## Parameters

### `mapflag`: int

The mapflags to set its value to. This must be a valid mapflags slot or an exception will be thrown. Since there are only 10 mapflags, valid values ranges from 0 to 9 inclusive.

### `value`: `true` | `false`

The value to set the map flag to. This must be a valid bool or an exception will be thrown.

### `flagvar`: int

The [flagvar](../../Flags%20arrays/flagvar.md) slot containing the mapflags slot. This must be a valid [flagvar](../../Flags%20arrays/flagvar.md) slot or an exception will be thrown. The value of the [flagvar](../../Flags%20arrays/flagvar.md) must be a valid mapflags slot or an exception will be thrown.

### `var`

The presence of this parameter indicate to operate in syntax (3). Its value does not matter, only its presence.

## Remarks

`map.mapflags` is an array of 10 bools that belongs to the current map, but is reset explicitly on the map's Start and unlike other flags arrays, this one is never saved on the [Save File](../../External%20data%20format/Save%20File.md). This gives it a very ephemeral and temporary usage because all their values are reset on every map loads.
