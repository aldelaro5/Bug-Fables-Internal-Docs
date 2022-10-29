# Var

Replaces the text from this command to the textual representation of a flagvar with left padding support or set a flagvar to a value.

## Syntax

(1)

````
|string,flagvar|
````

(2)

````
|single,flagvar,pad,totalwidth|
````

(3)

````
|single,flagvar,pad,totalwidth,padchar|
````

(4)

````
|single,flagvar,newvalue|
````

## Parameters

### `flagvar`:  int

The flagvar slot to get or set. This must corresponds to an `int` of a valid flagvar slot or an exception will be thrown.

### `pad`: `pad`

After obtaining the flagvar, this can optionally be specified to pad the beginning of the number with `padchar` such that the final length of the string is `totalwidth`. Any other value of this parameter will be ignored (with case sensitivity) and the behaviour will be like (4). If this is specified and `desiredlength` is not specified, an exception will be thrown.

### `totalwidth`: int

When `pad` is specified, the designed length of the string after padding. The value must a valid `int` value or an exception will be thrown.

### `padchar`: char

When `pad` is specified, the character to pad the text with. If this is not specified, the default value is `0`. If the length of this parameter exceeds 1, the first character is assumed to be the value.

### `newvalue`: int |  `v[ar]`int | `money`

Value to set the flagvar to: 

* `int`: Sets the flagvar in slot `flagvar` to this parameter's value. The value must a valid `int` value or an exception will be thrown.
* `v[ar]`int: Sets the flagvar in slot `flagvar` to the flagvar at the slot of the `int` after the `v` or `var` prefix. The `int` part must be a valid `int` value of a valid flagvar slot or an exception will be thrown. Any other prefix is not allowed and will cause an exception to be thrown.
* `money`: Sets the flagvar in slot `flagvar` to the current berry count (this is case sensitive).

## Remarks

With the exception of (4), this command will cause SetText to resume processing at the same character position to accommodate the text replacement of the input string at the position this command is being processed.
