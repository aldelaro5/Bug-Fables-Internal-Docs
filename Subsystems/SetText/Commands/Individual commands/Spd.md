# Spd

Set the wait time in seconds between letters.

## Syntax

````
|spd,interval|
````

## Parameters

### `interval`:  float

The time in seconds to wait between letters. This value must be a valid float or an exception will be thrown. If -1.0 is specified, the value defaults to 0.02.

## Remarks

This is an alias of [Speed](Speed.md).

By default, the wait time between letters is 0.02 seconds or 0.03 in `Japanese`. This command allows to change this. If a wait time of 0.0 is specified, it disables it entirely.

While this command works in [Dialogue mode](../../Dialogue%20mode.md) or not, in non dialogue mode, an `|spd,0|` command is prepended to the input string. This means that the default speed in non dialogue mode is effectively 0.0. It is still possible to override it with this command inside the input string itself.
