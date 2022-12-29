# Takeopenquests

Move all [BoardQuests](../../../Enums%20and%20IDs/BoardQuests.md) in the open board to the taken board with the exception of the 5 bounty ones.

## Syntax

````
|takeopenquests|
````

## Parameters

None.

## Remarks

This will not interact with the Bounty [BoardQuests](../../../Enums%20and%20IDs/BoardQuests.md) even if any is in the open board.

This command also handles placing and removing the NoQuest element from the open or taken board whenever applicable as well as enable each [BoardQuests](../../../Enums%20and%20IDs/BoardQuests.md)'s taken [flagvar](../../../Flags%20arrays/flagvar.md) slot.
