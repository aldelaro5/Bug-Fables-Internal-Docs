# Input ids
The game has a total of 10 inputs that are identified by a number from 0 to 9. The association of an input id to the actual action is defined by convention that is respected throughout the game and it is the only way for the game to know if a specific input is pressed or not.

Here is the table of the inputs ids to their matching action:

|Id|Name|Description|
|-:|----|-----------|
|0|Up|The up movement direction in the overworld or going up in UI menus|
|1|Down|The down movement direction in the overworld or going down in UI menus|
|2|Left|The left movement direction in the overworld or going left in UI menus|
|3|Right|The right movement direction in the overworld or going right in UI menus|
|4|Confirm / Jump / Interact|The primary button input used to confirm a UI menu option or for jumping or interacting in the overworld|
|5|Cancel / Field Action|The secondary button input used to cancel a UI menu option, activate [isholdingskip](../SetText/Related%20Systems/Text%20advance.md) or using a field ability in the overworld|
|6|Scroll Faster / Switch Party|The tertiary button input used to rotate the party in the overworld or battle or to page scroll in [ItemLists](../ItemList/ItemList.md) when held. It's also used to initiate a [text backtracking](../SetText/Related%20Systems/Backtracking.md)|
|7|Show / Hide HUD|The quarternary button input used to toggle the HUD or for changing pages when consulting the equipped medals in the PauseMenu's medals window|
|8|Pause|The input used to pause the game|
|9|Help|A input used for many miscellaneous actions such as starting the `map`'s tattle or [NPC](../Entities/NPCControl/NPC.md)'s tattle [SetText](../SetText/SetText.md) dialogue, unequipping all medals in the PauseMenu's medals window, show the world map in the PauseMenu's main window or access the secret menu from the StartMenu|
