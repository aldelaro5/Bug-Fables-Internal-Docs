# Keyboard
On the PC version of the game, the keyboard control scheme is always available and unlike [controllers](Controllers.md), it can't be disabled. One can always control the game using a keyboard at any time with or without controllers present.

Since a keyboard is very simple in concept where it comes to inputs, its setup process is much simpler than controllers. All bindings for all 10 [inputs](Inputs.md) are stored in InputIO.`keys` which is an array of 10 KeyCodes where each element contains the bound KeyCode indexed by input id. This array is persisted to the [config file](../External%20data%20format/Config%20File.md).

## Default bindings
These bindings have defaults which can be set using a method called SetDefaultKeys:

```cs
public static void SetDefaultKeys()
```
It sets `keys` to their default values which are the game's default keyboard bindings:

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

The method is normally only called towards the end of [LoadEssentials](../MainManager/Boot%20and%20reset%20process.md#loadessentials) and when pressing the F1 key in the keyboard binding window of the PauseMenu.

## Configuring the bindings
The only way to configure the bindings is through the keyboard binding window of the PauseMenu. Specifically, it involves an ItemList with the [keyboard binding listtype](../ItemList/List%20Types%20Group%20Details/Key%20Bindings%20List%20Type.md).

Not all KeyCodes are allowed to be configured under normal gameplay. The allowed ones are in InputIO.`bindingkeys` which is a hardcoded array. It contains all standard keys present on a full sized keyboard (including the numpad) except for any F keys and the Apple / Windows / meta keys.

## Rendering helpers
For display purposes, there is some logic in regards to displaying the name of a certain key to simplify it compared to its KeyCode enum value name. This is mainly done by ButtonIsLong:

```cs
public static string ButtonIsLong(int id)
```
It returns a friendly string corresponding to the bound keyboard KeyCode in `keys` whose [input](Inputs.md) id of `id`.

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

Additionally, there's another method that calls the one above, but with more logic:

```cs
public static bool LongButton(int id)
```
It returns true if ButtonIsLong(`id`) returns a string longer than 1 character while not containing `Arrow`, returns false otherwise. This is meant to tell if the rendering of the keyboard input's glyph needs to take more space than usual.

Finally, there's a method to simply return the KeyCode enum value string as is:

```cs
public static string KeyboardString(int id)
```
Returns `keys[id]`'s KeyCode enum value string.
