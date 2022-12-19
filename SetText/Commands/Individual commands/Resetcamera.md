# Resetcamera

Resets the camera to a default state.

## Syntax

````
|resetcamera|
````

## Parameters

None.

## Remarks

Specifically, this sets the following fields:

* `camspeed`: 0.1
* `camoffsetspeed`: 0.1
* `changecamspeed`: false
* `camanglechange`: false
* `camoffset2`: Vector3.zero
* `camanglespeed`: 0.1
* `camtargetpos`: null
* `camoffset`: The default one (0.0, 2.25, -8.25) or the one defined in the [Maps](../../../Enums%20and%20IDs/Maps.md)
* `camangleoffset`: The default one (10.0, 0.0, 0.0) or the one defined in the [Maps](../../../Enums%20and%20IDs/Maps.md)
* `camtarget`: null or the player if it exists.
