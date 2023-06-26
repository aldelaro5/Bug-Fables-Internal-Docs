# Openpause

Change the active window of the existing pause menu.

## Syntax

````
|openpause,windowid|
````

## Parameters

### `windowid`: int

The window id of the pause menu to open to. This must be a valid int or an exception will be thrown. Here are the different window id available:

|Id|Description|
|--|-----------|
|-1|No window shown|
|0|Top level pause window (only offers "Inventory" and "Medals and Stats" during a battle)|
|1|"Inventory" window|
|2|"Medals and Stats" window|
|3|"Library" window|
|4|"Settings" window|
|5|Key Bindings's window|
|6|Map of Bugaria's window|
|7|Controller Bindings's window|

Any other value has undefined behaviors.

## Remarks

This command does nothing if the pause menu isn't present and it also sets [End](End.md) to true.

This command is used in the very specific case where the pause menu is already opened, but navigation to a different window needs to happen via SetText. Its main application in the game is when returning to the pause menu after a key item usage prompt occurred.
