# Inputs
This game has 2 input scheme:

- [Keyboard](Keyboard.md) (always enabled and available)
- [Controllers](Controllers.md) (enabled by default, but it can be disabled and it is optional)

These schemes can be used to address a total of 10 game inputs that are identified by a number from 0 to 9. The association of an input id to the actual action is defined by convention that is respected throughout the game and it is the only way for the game to know if a specific input is pressed or not.

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

MainManager has an enum called `Directions` that contains the same values as the table above, but in most of the game, the int form of the value is used.

## Input aliases
MainManager.GetKey also supports 4 inputs aliases which are negative input ids that refers to an aggregations of inputs. It is mentioned here because it is by far the most used method by the game to pull inputs.

Here is a table with the supported aliases:

|Id|Description|
|-:|-----------|
|-4|Any game inputs (id of 0 to 9)|
|-3|Any button inputs (id of 4 to 9)|
|-2|Any axis inputs (id of 0 to 3)|
|-1|Any inputs detected by Unity's Input API (Input.anyKey / Input.anyKeyDown)|

## Joystick inputs
There is a variation to the input id system where the first 4 inputs have a different meaning when talking about input bindings for joysticks. These are called joystick input ids and they specifically concerns bindings [controllers](Controllers.md).

It only changes the meaning of the first 4 inputs which are axis based, but it doesn't change any other inputs which are the buttons based ones. This is done because the way axis based inputs works on controllers varies slightly since for controllers, there's only 2 axis (vertical and horizontal), but offered in 2 flavors by the game (digital and analogue). The game only uses this form of ids when addressing controller bindings specifically, but there are abstractions available by the game to translate input ids to joystick input ids when using controllers.

Here are the new mapping for the first 4 input values under this id scheme:

|Id|Name|
|-:|----|
|0|Analogue Horizontal|
|1|Analogue Vertical|
|2|D-Pad Horizontal|
|3|D-Pad Vertical|

## Input related methods in InputIO
The following methods in InputIO concerns inputs of any kinds whether [Keyboard](Keyboard.md) or [controllers](Controllers.md) are used.

```cs
public static bool anyKey()
```
Returns Input.anyKey.

```cs
public static bool anyKeyDown()
```
Returns Input.anyKeyDown.

```cs
public static bool GetKey(int key, bool joy)
```
If `IsConsole` is true, this always returns false.

Otherwise, the return value depends on `joy`:

- If it's false, it returns Input.GetKey(`keys[key]`)
- If it's true, it returns Input.GetKey(`joykeys[key]`)

```cs
public static bool GetKeyDown(int key, bool joy)
```
If `IsConsole` is true, this always returns false.

Otherwise, the return value depends on `joy`:

- If it's false, it returns Input.GetKeyDown(`keys[key]`)
- If it's true, it returns Input.GetKeyDown(`joykeys[key]`)

```cs
public static bool GetKeyRaw(KeyCode key, bool hold)
```
This method is UNUSED because it is never called.

Returns GetKey(`key`) if `hold` is true or returns GetKeyDown(`key`) if `hold` is false.
