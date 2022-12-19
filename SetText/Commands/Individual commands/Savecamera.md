# Savecamera

Saves the current camera's configuration and limits for retrieval later using [Loadcamera](Loadcamera.md).

## Syntax

````
|savecamera|
````

## Parameters

None.

## Remarks

Specifically, it saves the following fields using `SaveCameraPosition(true)`:

* `tempcampos` is set to `camtargetpos`
* `tempcamoffset` is set to `camoffset`
* `tempcamspeed` is set to `camspeed`
* `tempcamangleoffset` is set to `camangleoffset`
* `tempcamtarget` is set to `camtarget`
* `tempmaplp` is set to `map.camlimitpos`
* `tempmapln` is set to `map.camlimitneg`

They can be loaded back to their saved value using [Loadcamera](Loadcamera.md), but it is also possible to do the same by calling `SaveCameraPosition(false)`.
