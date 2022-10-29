# Anstring

Replaces the text from this command to a medal or item's prepender string from the game data.

## Syntax

(1)

````
|string,id|
````

(2)

````
|single,id,ismedal|
````

## Parameters

### `id`:  int | `var`int

The item (1) or medal (2) id to obtain the prepend string from:

* `int`: The item (1) or medal (2) id. This must be a valid `int` value of a valid item (1) or medal (2) id or an exception will be thrown.
* `var`int: The flagvar slot that contains the item (1) or medal (2) id. The `int` part must be a valid `int` value of a valid flagvar slot or an exception will be thrown. Any other prefix is not allowed and will cause an exception to be thrown. The flagvar must contain a valid item (1) or medal (2) id or an exception will be thrown.

### `ismedal`: int

When a non zero value is specified, `id` refers to a medal id. If the value is 0 or not specified, `id` is an item id. This must corresponds to a valid `int` value or an exception will be thrown.

## Remarks

This command will cause SetText to resume processing at the same character position to accommodate the text replacement of the input string at the position this command is being processed.
