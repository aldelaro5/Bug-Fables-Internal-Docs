# Addquest

Add a [BoardQuests](../../../Enums%20and%20IDs/BoardQuests.md) to the open board or move an existing [BoardQuests](../../../Enums%20and%20IDs/BoardQuests.md) to another board.

## Syntax

(1) (Add the quest to the open board)

````
|addquest,quest|
````

(2)

````
|addquest,quest,newboard|
````

## Parameters

### `quest`: int

The [BoardQuests](../../../Enums%20and%20IDs/BoardQuests.md) to add or move board. This must be a valid [BoardQuests](../../../Enums%20and%20IDs/BoardQuests.md) id or an exception will be thrown.

### `newboard`: int

The board to move the `quest` to. This must be 0 (open), 1 (taken) or 2 (completed) or an exception will be thrown.

## Remarks

If syntax (2) is used with the `newboard` being taken or completed, this will also show the journal updated HUD.

For all syntaxes, this handles the removal of NoQuest to the older board if applicable.
