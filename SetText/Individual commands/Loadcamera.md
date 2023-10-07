# Loadcamera

Loads the saved camera's configuration and limits saved by [savecamera](Savecamera.md).

## Syntax

````
|loadcamera|
````

## Parameters

None.

## Remarks

Specifically, it loads the following fields using `SaveCameraPosition(false)`:

* `camtargetpos` is set to `tempcampos`
* `camspeed` is set to `tempcamspeed`
* `camoffset` is set to `tempcamoffset`
* `camangleoffset` is set to `tempcamangleoffset`
* `camtarget` is set to `tempcamtarget`
* `map.camlimitpos` is set to `tempmaplp`
* `map.camlimitneg` is set to `tempmapln`
