```cs
public static string ButtonIsLong(int id)
```
Returns a friendly string corresponding to the bound keyboard KeyCode in `keys` whose [input](Inputs.md) id of `id`.

Here's a table showing each KeyCode and the string the method returns (the first row that applies is the one used, they are tested in order):

|KeyCode|Result string|
|-------|-------------|
|Any whose name is 1 character long|The KeyCode enum value name itself|
|LeftArrow or RightArrow|The KeyCode enum value name itself|
|LeftBracket|`[`|
|RightBracket|`]`|
|Comma|`,`|
|Backslash|`\`|
|KeypadMultiply|`*`|
|LeftControl|`L.Ctrl`|
|RightControl|`R.Ctrl`|
|Quote|`'`|
|BackQuote|A ` character|
|Return or KeypadEnter|`Enter`|
|Period or KeypadPeriod|`.`|
|Slash or KeypadDivide|`/`|
|Plus or KeypadPlus|`+`|
|Minus or KeypadMinus|`-`|
|Anything not mentioned above containing `Keypad` in the enum value name|The enum value name with `Keypad` removed|
|Anything not mentioned above containing `Alpha` in the enum value name|The enum value name with `Alpha` removed|
|Anything not mentioned above containing `Left` in the enum value name|The enum value name with `Left` replaced with `L.`|
|Anything not mentioned above containing `Right` in the enum value name|The enum value name with `Left` replaced with `R.`|
|Anything not mentioned above|The KeyCode enum value name itself| 

```cs
public static bool LongButton(int id)
```
Returns true if ButtonIsLong(`id`) returns a string longer than 1 character while not containing `Arrow`, returns false otherwise. This is meant to tell if the rendering of the keyboard input's glyph needs to take more space than usual.

```cs
public static bool GetKeyRaw(KeyCode key, bool hold)
```
This method is UNUSED because it is never called.

Returns GetKey(`key`) if `hold` is true or returns GetKeyDown(`key`) if `hold` is false.

```cs
public static void StartUp()
```
This method is called early during the [boot process](../MainManager/Boot%20and%20reset%20process.md), but the logic within depends on the specific store platform the game uses.

On the Steam version, `steam` is set to the SteamManager component of a newly created GameObject named `Steam Manager` that is rooted to the scene. This will initialise Steamworks.NET to enable Steam's achievements functionality.

```cs
public void OnApplicationQuit()
```
The logic of this Unity event depends on the specific store platform the game uses.

On the Steam version, SteamAPI.Shutdown is called which decomissions Steamworks.NET.

```cs
public static void Achivement(int id)
```
The logic of this Unity event depends on the specific store platform the game uses.

On the Steam version, it grants the `achivements[id]` (using SteamUserStats.SetAchievement) if it wasn't already (checked via SteamUserStats.GetAchievement) through Steamworks.NET. This is done only if SteamManager got initialised sucessfully, SteamUtils.IsOverlayEnabled returns true and SteamUserStats.RequestCurrentStats also returns true. If the achievement is granted, SteamUserStats.StoreStats is called.

This is an optional part of the [achievement system](../General%20systems/Achievements.md) so this method is only called when getting an in game achievement.

```cs
public static bool anyKey()
```
Returns Input.anyKey.

```cs
public static bool SaveExists(int id)
```
Returns whether or not the `saveX.dat` [save file](../MainManager/Methods/Save%20file.md) exists where `X` is `id`.

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
public static string KeyboardString(int id)
```
Returns `keys[id]`'s KeyCode enum value string.

```cs
public static int GetJoystickRaw()
```
Performs the following to obtain the first joystick button pressed or axises touched:

- Goes through all 20 `JoystickButton` KeyCode and calls Input.GetKeyDown with it. The first one that returns true will cause the method to return its number at the end of the enum value name (so the number from 0 to 19). If none of them were true, the method continues
- Goes through all 10 virtual axises and if any of their Input.GetAxis value are more than 0.15 away from 0.0, -X is returned where X is the axis id + 1
- If none of the axises satisfy the condition above, -55 is returned to signal no buttons or axises were detected as being pressed or touched

This method is mainly used for controller bindings to detect the physical input used.

```cs
public static void GetNeutrals()
```
This is practically UNUSED because it is only called as part of the UNUSED controller binding window of the PauseMenu.

Reset all `neutrals` values to their matching Input.GetAxis() values (mapped by index since all axis's names are numbers from 1 to 10).

```cs
public static float JoyStick(int id)
```
Returns the axis value bound to the [input](Inputs.md) id `id` given the current controller settings and MainManager.`joyid`. Since this only handles axis inputs, `id` must be between 0 and 3 as other bindings are buttons based.

If MainManager.`usejoystick` is 5 (custom bindings), the entire method logic is overriden to ignore MainManager.`joyid`. In that case, the return value is Input.GetAxis(MainManager.`joybinds[id]`.ToString()) which returns the bound axis configured in the user's bindings.

For any other settings, the return value is Input.GetAxis(`X`) where `X` is a number from 1 to 10 and corresponds to the physical axis number of the controller. The exact axis ids used depends on MainManager.`joyid` which tells the preset mapping to use.

What follows is all the MainManager.`joyid`'s bindings grouped by possible values (some of these have the same bindings between different values). They are harcoded in the game from supposedly good mappings.

## 0 / 8 - Xbox 360 / Xbox Linux

|[input](Inputs.md) id|Axis id|
|-------------------:|---------|
|0 (up)|1|
|1 (down)|2|
|2 (left)|6|
|3 (right)|7|

## 1 - Dualshock 4

|[input](Inputs.md) id|Axis id|
|-------------------:|---------|
|0 (up)|1|
|1 (down)|2|
|2 (left)|7|
|3 (right)|8|

## 2 / 3 / 4 - Switch Pro Controller / Switch Joy-Con L / Switch Joy-Con R

|[input](Inputs.md) id|Axis id|
|-------------------:|---------|
|0 (up)|2|
|1 (down)|4|
|2 (left)|9|
|3 (right)|10|

## 5 / 6 / 7 - Generic Controller 1 / Generic Controller 2 / Fight Pad Pro for Switch

|[input](Inputs.md) id|Axis id|
|-------------------:|---------|
|0 (up)|1|
|1 (down)|2|
|2 (left)|5|
|3 (right)|6|

```cs
public static string ReadFile(string path)
```
Returns File.ReadAllText(`path`) unless the file doesn't exist where null is returned instead.

```cs
public static bool DeleteFile(string path)
```
Deletes the file at `path` and returns true if it was sucessful.

The method returns false under any of the following conditions:

- The file doesn't exist
- Any exception was thrown when deleting the file

```cs
public static void SetDefaultKeys()
```
Set `keys` to their default values which are the game's default keyboard bindings:

|[input](Inputs.md) id|KeyCode value|
|--------------------:|------------|
|0|UpArrow|
|1|DownArrow|
|2|LeftArrow|
|3|RightArrow|
|4|C|
|5|X|
|6|Z|
|7|V|
|8|Escape|
|9|Return|

```cs
public static bool CreateFile(string path, string content)
```
Creates or opens a file at `path`, write `content` into it, close it and returns true to indicate sucess.

The method returns false if any exception is thrown during this process.

```cs
public static bool LoadSettings(bool overwrite)
```
Either read [config.dat](../External%20data%20format/Config%20File.md) if it exists and `overwrite` is false followed by applying the settings read or create a new config file if it doesn't exist or `overwrite` is true and saves the current field settings on it. In either cases, true is returned to indicate success.

Specifically, reading an existing file involves:

- MainManager.ReadSettings(File.ReadAllLines(`config.dat`))
- MainManager.ApplySettings()

Both method are documented in the [config file](../External%20data%20format/Config%20File.md) documentation. If any exception is thrown during this process, LoadSettings is recalled with an `overwrite` of true to force the recreation of the file so the game settings and the file are ensured to be in sync. If no exceptions occured, true is returned to indicate success.

In the case of creating a new `config.dat`, it involves the following:

- MainManager.SaveSettings is called to get the text to write to the config file
- File.CreateText(`config.dat`) called
- The string obtained earlier is written to the file
- The file is closed

If any exception is thrown during this process, a debug log of `config creation failed` happens and false is returned.

If no exception happened and `overwrite` was false (meaning an attempt to open the file resulted in its creation as it didn't exist), the resolution of the game is set to 1024 x 576 windowed.

```cs
public static bool Save(Vector3? savepos)
```
Writes the current game state to the current save file using `savepos` as the position of the party to save (if it's null, the current `player` position is used instead).

The logic to write to the save file is complex, here are the details:

- Obtain the [Save file](../MainManager/Methods/Save%20file.md) string data by calling MainManager.SaveFile(`savepos`)
- Creates or opens the `saveXt.dat` file where `X` is MainManager.`saveslot`. This is a temporary file that will be renamed later
- Call Encrypt on the SaveFile string obtained earlier to encode it
- Writes the encoded version of the string to the opened file
- Closes the file
- If the `saveX.dat` file exists where `X` is MainManager.`saveslot` (meaning the current save slot has had a backing file):
    - If the `saveXbackup.dat` file exists where `X` is MainManager.`saveslot`, it is deleted
    - The existing `saveX.dat` file is renamed to `saveXbackup.dat` which moves the existing save file to its backup one for redundancy purposes
- The `saveXt.dat` file that was just created is renamed to remove the `t` which fully commits the save file to disk
- True is returned to indicate success

If any exception is thrown at any point in this process, false is returned alongside a Debug log saying the exception's Message prefixed with `save exception: `.

```cs
public static string Encrypt(string input)
```
Returns a string where every character in `input` was XORed with the int contained in the `Data/key` TextAsset. This is used to encode or decode a [save file](../MainManager/Methods/Save%20file.md) before reading it or writting it.

```cs
public static void GetJoyButtons()
```
Assigns all `joykeys` elements given the current MainManager.`joyid` value and current controller settings. Essentially, all digital button [input](Inputs.md) bindings are updated to reflect the current settings and controller preset selected. This doesn't manage the first 4 inputs because they are axis based, not buttons based.

If MainManager.`usejoystick` is 5 (custom bindings), the entire method logic is overriden to ignore MainManager.`joyid`. In that case, all `joykeys` elements are set to their MainManager.`joybinds` equivalent JoystickButton KeyCode. For example, `joykeys[0]` is set to JoystickButtonX where X is MainManager.`joybinds[4]` (`joykeys`'s elements do not contain bindings for the first 4 inputs). However, if the matching MainManager.`joybinds` element is -55 (no binding), the KeyCode set to the `joykeys` element is None.

For any other settings, all `joykeys` are assigned given the current MainManager.`joyid`, but with a variation. The KeyCodes assigned contains a prefix that can vary depending on if MainManager.`usejoystick` is 3 (choose manually) or not. If it is, the prefixes are JoystickXButton where X is MainManager.`forcejoystick` which will force to check a specific joystick connected with caveats (see the section below for details). If it's not set to choose manually, the prefixes are JoystickButton which means all controllers connected are checked at once for the specific physical button.

What follows is all the MainManager.`joyid`'s bindings grouped by possible values (some of these have the same bindings between different values). They will be expressed in input id so to get the equivalent `joykeys` element index, subtract that number by 4. All button ids are 0 indexed and matches the suffix of the KeyCode element. They are harcoded in the game from supposedly good mappings.

## 0 / 8 - Xbox 360 / Xbox Linux

|[input](Inputs.md) id|Button id|
|-------------------:|---------|
|4 (confirm)|0|
|5 (cancel)|1|
|6 (switch)|2|
|7 (toggle HUD)|3|
|8 (pause)|7|
|9 (help)|6|

## 1 - Dualshock 4

|[input](Inputs.md) id|Button id|
|-------------------:|---------|
|4 (confirm)|1 (2 instead if [languageid](../SetText/languageid.md) is `Japanese`)|
|5 (cancel)|2 (1 instead if [languageid](../SetText/languageid.md) is `Japanese`)|
|6 (switch)|0|
|7 (toggle HUD)|3|
|8 (pause)|9|
|9 (help)|8 (only assigned if Application.platform isn't PS4)|

## 2 - Switch Pro Controller

|[input](Inputs.md) id|Button id|
|-------------------:|---------|
|4 (confirm)|1|
|5 (cancel)|0|
|6 (switch)|3|
|7 (toggle HUD)|2|
|8 (pause)|9|
|9 (help)|8|

## 3 - Switch Joy-Con L

|[input](Inputs.md) id|Button id|
|-------------------:|---------|
|4 (confirm)|1|
|5 (cancel)|0|
|6 (switch)|3|
|7 (toggle HUD)|2|
|8 (pause)|8|
|9 (help)|5|

## 4 - Switch Joy-Con R

|[input](Inputs.md) id|Button id|
|-------------------:|---------|
|4 (confirm)|1|
|5 (cancel)|0|
|6 (switch)|3|
|7 (toggle HUD)|2|
|8 (pause)|9|
|9 (help)|5|

## 5 / 6 - Generic Controller 1 / Generic Controller 2

|[input](Inputs.md) id|Button id|
|-------------------:|---------|
|4 (confirm)|0|
|5 (cancel)|1|
|6 (switch)|3|
|7 (toggle HUD)|4|
|8 (pause)|11|
|9 (help)|10|

## 7 - Fight Pad Pro for Switch

|[input](Inputs.md) id|Button id|
|-------------------:|---------|
|4 (confirm)|2|
|5 (cancel)|1|
|6 (switch)|3|
|7 (toggle HUD)|0|
|8 (pause)|9|
|9 (help)|8|
