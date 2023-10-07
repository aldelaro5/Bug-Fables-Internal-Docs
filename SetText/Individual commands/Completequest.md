# Completequest

Sets a [BoardQuest](../../Enums%20and%20IDs/BoardQuests.md) as completed which removes it from the taken board and add it to the completed board.

## Syntax

````
|completequest,questid|
````

## Parameters

### `questid`: int

The [BoardQuest](../../Enums%20and%20IDs/BoardQuests.md) id to mark as completed. This must be a valid [BoardQuest](../../Enums%20and%20IDs/BoardQuests.md) or an exception will be thrown.

## Remarks

This command will also ensure that the `NoQuest` value is removed from the completed board and added to the taken board if no quests are taken after the transfer. It will also increment [flagvar](../../Flags%20arrays/flagvar.md) 47 and show the quest complete HUD message for 350 frames.

It should be noted this command will still work even if the [BoardQuest](../../Enums%20and%20IDs/BoardQuests.md) isn't in the taken board. In that case, it will be added to the completed board.
