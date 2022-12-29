# Camlimit

Set the camera's positional lower bound limits (this command is likely broken, see remarks for details)

## Syntax

````
|camlimit,dummy,dummy,dummy,lowerx,lowery,lowerz|
````

## Parameters

### `dummy`: float

Anything in here is ignored due to an issue with this command's processing. The value must still be a valid float or an exception will be thrown.

### `lowerx`: float

The X coordinate of the lower bound limit to set the camera. This must be a valid float or an exception will be thrown.

### `lowery`: float

The Y coordinate of the lower bound limit to set the camera. This must be a valid float or an exception will be thrown.

### `lowerz`: float

The Z coordinate of the lower bound limit to set the camera. This must be a valid float or an exception will be thrown.

## Remarks

This command was presumably supposed to set the lower and upper bounds of the camera's positional limit, but due to a bug, it only sets the lower bound using the last 3 parameters. This makes it ignore the first 3.

This command is unused under normal gameplay.
