# Controllers
This game supports a wide variety of controllers through the Unity's legacy input system. Unlike [keyboard](Keyboard.md) control, controllers support is optional and can be disabled with a setting. If enabled, it allows to control the game alongside a keyboard, the glyphs rendered with any ButtonSprite will adjust accordingly.

However, controllers are much more complex conceptually than a keyboard when it comes to handling their input. While the game is generally well equipped to handle them, it's important to understand how it works at the engine level before explaining what role InputIO does to simplify this further. This is documented here because unfortunately, Unity's documentation is lacking when it comes to explain how the engine abstracts input handling at the OS level. Since this game's PC version is only available on Windows, this will only document the specifics for Unity on Windows.

## How controllers work in Unity
On Windows, Unity's legacy input system makes use of 2 well known Windows specific input API:

- RawInput: A very stable windows API that is essentially the oldest input API offered by Windows. Most controllers that supports Windows in general works using this API. It is not to be confused with DirectInput which is a deprecated API Unity doesn't use, but interestingly, DirectInput is an abstraction over RawInput. So while their functionality might be close, they aren't the same as RawInput tends to be more performant and unlike DirectInput, it is stable without plans for its depreciation
- XInput: A much simpler input API, but it is limited to supported hardware. XInput devices mostly includes Xbox branded controllers, but it is possible to see third party devices that natively supports it and there's also third party emulation tools available to simulate it (such as Steam's own Steam Input). While all XInput devices technically supports RawInput, it is prefered to use XInput when it is available because XInput has specific knowledge on how these devices work which RawInput does not have. A well known example is an Xbox controller on RawInput will expose both of its triggers as a single axis, but XInput is able to expose both triggers's axises separately

The basic summary is that most input handling will go through RawInput, but if a device is found to be XInput compatible, its handling will be done via XInput instead. If the device isn't compatible with XInput, RawInput is used.

However, it's possible in some cases that a device supports neither of these APIs. To test this, one can use Windows's built in gamepad tester which can be launched by typing `joy.cpl` in the start menu. It will show a list of connected controllers and when consulting its properties, it will show a test windows showing which buttons are pressed and the value of the axises of the device.

If every buttons and every axises causes a reaction in this window, the controller can be supported by the game (but manual bindings might be necessary). However, if no reaction happens at all or the reactions are eratic, the device won't work as is. In such case, it might be preferable to use an emulation layer, the most widely used one is Steam's Steam Input which provides a wide support for different controllers.

This is how Unity handles controllers at the OS level, but it also abstracts this into a simpler interfaces using virtual axises and joystick related KeyCode values. Let's go into what Unity does to abstract this.

## Unity's Input API
While Unity's Input API is meant to work multiplatform, this section will detail the Windows specific details.

There are 2 key concept the Unity's Input API introduces:

- Joystick related KeyCode values
- Virtual axises

### Joystick KeyCode values
The first concept is easy to explain: the KeyCode enum isn't limited to represent just [keyboard](Keyboard.md) related inputs, it also contains values that represents physical controller buttons. There are several values starting from 330 to 509, but all of them have their enum value name fit the following format:

`JoystickXButtonY` where:

- `X` is empty for any connected controllers or a number between 1 and 8 to refer to a specific connected controller
- `Y` is the physical button id of the controller reffered to by `X` which is a number between 0 and 19. Interestingly, this number matches the one that can be seen light up using `joy.cpl` for the button, except that Unity's numbers starts by 0 while `joy.cpl` numbers starts by 1

For the most part, the game prefers to use an `X` of empty to refer to any connected controllers, but there is a setting that allows to refer to a specific controller. Unfortunately, this setting has many caveats, more details in a section below.

### Virtual axises
Unity allows to configure a variable amount of axises which reports a float value when calling Input.GetAxis using its id configured (which is a string).

All of them are configured at the project level and while they can offer many diverse features such as emulating certain inputs, Bug Fables configures them in a very specific way to simplify it further. There are 10 virtual axises in the game and they are all configured like the following (any fields not mentioned are left blank):

- Id: Number between 1 and 10 as a string depending on the specific axis
- Gravity: 3.0
- Dead: 0.2
- Sensitivity: 1.0
- Snap: Yes
- Invert: No
- Type: Joystick Axis
- Axis: The matching one depending on the Id field (Id 1 is labeled as the "X axis" and Id 2 is labeled as the "Y axis" and the rest are labeled "Xth axis" where X is the Id)
- Joy Num: Get Motion from all Joysticks

This specific configuration leads to the game being able to address any of the first 10 axises defined on any connected controllers. In that sense, the id string of the axis will match the physical axis id of the controller. While it is much closer to the hardware than how virtual axises are normally used, it allows to address them more liberally and increase the amount of supported controllers. The game only makes use of 4 axises (vertical and horizontal, each in analogue and digital flavors) so as along as these axises are defined within the first 10 physical ones, the controller can work with the game.

## Game's controller input pipeline
Finally, we have the game's abstraction of Unity's Input API which is contained in InputIO and a part of MainManager. This section will focus on the InputIO part and the settings configurable from the [config file](../External%20data%20format/Config%20File.md).

There is a specific pipeline to determine which KeyCode / virtual axis corresponds to which game input. This pipeline is configurable using fields from the [config file](../External%20data%20format/Config%20File.md). These fields are as follows:

- MainManager.`usejoystick`: The primary controller setting field
- MainManager.`forcejoystick`: The secondary controller setting field
- MainManager.`joybinds`: Specific controller bindings used when using custom bindings

Regardless of the values of these fiels, the overall structure of the pipelines remains similar. Here's the general procedure of the pipeline:

- At the end of every MainManager.Update calls once `basicload` is true (after [LoadEverythin](../MainManager/Boot%20and%20reset%20process.md#loadeverything-part-12)),  MainManager.[GetJoystick](../MainManager/Methods/Inputs.md#getjoystick) is always called
- If controllers aren't disabled, when any controller is being used while the [keyboard](Keyboard.md) was used prior, MainManager.GetJoystick determines the value MainManager.`joyid` should have according to controller settings. After, InputIO.GetJoyButtons is called
- InputIO.GetJoyButtons determines all the InputIO.`joykeys` KeyCode values acording to current settings and the new MainManager.`joyid` value. From now on, any button [input](Inputs.md) (id 4 to 9) are bound using the InputIO.`joykeys` array where each element corresponds to the KeyCode bound of an input where the ids are shifted by 4 (so input id 4 is index 0, input id 5 is index 1, etc...)
- Any axis inputs are processed by InputIO.JoyStick which takes a [joystick input](Inputs.md#joystick-inputs) and handles it if it's an axis input id (0 to 3). The axis to read is determined acording to current settings and the new MainManager.`joyid` value

It means that InputIO handles the actual bindings with 2 methods while MainManager.GetJoystick updates the bindings configurations. Here are the InputIO methods signatures:

```cs
public static float JoyStick(int id)
```
Returns the axis value bound to the [joystick input](Inputs.md#joystick-inputs) id `id` given the current controller settings and MainManager.`joyid`. Since this only handles axis inputs, `id` must be between 0 and 3 as other bindings are buttons based.

```cs
public static void GetJoyButtons()
```
Assigns all `joykeys` elements given the current MainManager.`joyid` value and current controller settings. Essentially, all digital button [input](Inputs.md) bindings are updated to reflect the current settings and controller preset selected. This doesn't manage the first 4 inputs because they are axis based, not buttons based.

Now that we have a basic understanding of the input pipeline, let's look into the concept of presets and how the 3 controller settings changes the pipeline.

### Controller binding presets
In most pipeline configuration, there is a concept of a controller preset. There are 9 in the game and they contain hardcoded bindings of a known controller configuration. The current preset used is stored in MainManager.`joyid`.

MainManager.`joyid` selects the controller preset. It dictates the bindings to use by InputIO.GetJoyButtons / InputIO.JoyStick (assuming custom bindings aren't used) and it also controls the glyph rendering of all ButtonSprite.

Here are the possible values and their bindings where all bindings are expressed in [joystick input](Inputs.md#joystick-inputs) id where all axis inputs shows their virtual axis id and all button inputs shows their KeyCode button id (as explained above in the Unity Input section):

|Preset|English name|0 (Analogue Vertical)|1 (Analogue Horizontal)|2 (D-Pad Vertical)|3 (D-Pad Horizontal)|4 (Confirm)|5 (Cancel)|6 (Switch Party)|7 (Toggle HUD)|8 (Pause)|9 (Help)|
|-----:|----|-|-|-|-|-|-|-|-|-|-|
|0|Xbox 360|1|2|6|7|0|1|2|3|7|6|
|1|Dualshock 4|1|2|7|8|1<sup>*</sup>|2<sup>*</sup>|0|3|9|8<sup>**</sup>|
|2|Switch Pro Controller|2|4|9|10|1|0|3|2|9|8|
|3|Switch Joy-Con L|2|4|9|10|1|0|3|2|8|5|
|4|Switch Joy-Con R|2|4|9|10|1|0|3|2|9|5|
|5|Generic Controller 1|1|2|5|6|0|1|3|4|11|10|
|6|Generic Controller 2|1|2|5|6|0|1|3|4|11|10|
|7|Fight Pad Pro for Switch|1|2|5|6|2|1|3|0|9|8|
|8|Xbox Linux|1|2|6|7|0|1|2|3|7|6|

*: These 2 bindings are swapped with each other if the MainManager.[languageid](../SetText/languageid.md) is `Japanese`

**: This binding is only assigned if Application.platform isn't PS4

Additionally, if the pipeline is configured to do so, MainManager.GetJoystick will automatically determine the value of MainManager.`joyid` depending on a connected controller's name. Here is an ordered table that shows each preset's detection criteria (only the first one mentioned applies, any presets not present can't be auto detected by MainManager.GetJoystick):

|Preset|English name|Controller name detection criteria|
|-----:|----|--------------------------------|
|7|Fight Pad Pro for Switch|Name contains `Fight Pad Pro`|
|6|Generic Controller 2|Name is exactly `Bluetooth Gamepad   ` (with 3 trailing spaces) or it contains `PC/PS3/Android`|
|5|Generic Controller 1|Name contains `ðñò`|
|2|Switch Pro Controller|Name is exactly `Wireless Gamepad`|
|1|Dualshock 4|Name doesn't contain `Xbox` or `XBOX`, but it has to contain `Wireless`, `PS4`, `Sony`, `tation`, `PLAYSTATION` or `3 TURBO`|
|0|Xbox 360|None of the above applies (even if the name doesn't contain `Xbox` or `XBOX`, the value falsback to this)|

### Controler input settings
The primary field (MainManager.`usejoystick`) is the one that controls the most about the pipeline. Here are the possible values and their English names:

- 0: DISABLED
- 1: AUTO DETECT
- 2: FORCE XBOX
- 3: CHOOSE MANUALLY
- 4: PRE-CONFIGURED
- 5: CUSTOM BINDINGS

These can be considered the "mode" the pipeline operates on. It dictates the specific meaning of MainManager.`forcejoystick` and whether or not MainManager.`joybinds` is used. Most notably, it tells how MainManager.`joyid` is determined by MainManager.GetJoystick and how it is used by InputIO.GetJoyButtons / InputIO.JoyStick so it controls how the controller preset is selected and where it is used.

Here's a table that summarises all the pipeline configurations for each value of MainManager.`usejoystick`:

|Value|Mode English name|MainManager.`forcejoystick` semantic|MainManager.`joyid` value and usage|MainManager.`joybinds` usage|Description|
|-----:|---------|------------------------------------|----------------------------------|---------------------------|-----------|
|0|DISABLED|Not used|Not used|Not used|MainManager.GetJoystick completely ignores controllers and the entire controller input pipeline is disabled|
|1|AUTO DETECT|Not used|Automatically determined by MainManager.GetJoystick according to the last connected controller's name and it is used both for input bindings and for glyph rendering|Not used|The default mode where the game will try to guess the value of MainManager.`joyid` given the controller's name and process inputs with this guess|
|2|FORCE XBOX|Not used|The value is set to 0 (Xbox 360) by MainManager.GetJoystick and it is used both for input bindings and glyph rendering|Not used|MainManager.`joyid` is forced to always be 0 which is Xbox 360 so all inputs are processed assuming this preset. It is practically the same than using PRE-CONFIGURED with a MainManager.`forcejoystick` of 0|
|3|CHOOSE MANUALLY|The physical controller id indexed from the return of InputIO.Controllers or -1 if it is set to NONE|The same as AUTO DETECT, but instead of always using the last connected controller, the controller used is the one selected by MainManager.`forcejoystick` and other controllers are ignored|Not used|The same as AUTO DETECT, but the physical controller used is chosen manually instead of always being the last connected one. If MainManager.`forcejoystick` isn't -1 (NONE), it also changes the KeyCode InputIO.GetJoyButtons uses to assign the InputIO.`joykeys` elements such that it uses the joystick specific KeyCode instead of the ones that refers to any joysticks. NOTE: This setting is very flawed and broken, more details in a section below|
|4|PRE-CONFIGURED|The MainManager.`joyid` to use|The value is set to MainManager.`forcejoystick` by MainManager.GetJoystick and it is used for both input bindings and glyph rendering|Not used|Forces the controller binding preset to use and process inputs assuming the preset. Setting MainManager.`forcejoystick` to 0 (Xbox 360) is practically the same as using FORCE XBOX|
|5|CUSTOM BINDINGS|The MainManager.`joyid` to use|The value is set to MainManager.`forcejoystick` by MainManager.GetJoystick, but it is only used for glyph rendering|Used and it overrides the entire input binding of InputIO.GetJoyButtons / InputIO.JoyStick so MainManager.`joyid` is ignored when it comes to input bindings|Tells the pipeline to override all bindings and glyph rendering to the values specificed in the setting. The settings values controls the entire bindings and there are no controller bindings preset used|

#### Problems with choose manually mode
This mode can be considered broken because it doesn't work as it is supposed to.

What it was supposed to do was a more precise version of AUTO DETECT where it would work the same, but allow the player to select which connected controller to use instead of assuming the last one. This could have allowed to reduce the ambiguity and bind a specific controller. This is done by indexing the array returned by MainManager.Controllers which is essentially Input.GetJoystickNames, but each element with a length of 0 or 1 are removed. The index to use is given by MainManager.`forcejoystick` which is one of the configurable settings.

However, the concept and implementation are flawed for these reasons:

- It relies on the Unity's controllers list which can change with each device changes while the setting relies on the controller list not changing between each session
- While buttons inputs uses the joystick specific KeyCode values for the bindings when using the CHOOSE MANUALLY mode, the same can't be said for virtual axises which are always bound to any joystick's physical axises
- Setting MainManager.`forcejoystick` to -1 (NONE) isn't the same than setting MainManager.`usejoystick` to DISABLED and it will actually lead to unstable behaviors (such as enabling controllers, but using the previous assigned value of MainManager.`joyid` instead of setting it properly) and it will also lead to the setting not being honered by InputIO.GetJoyButtons
- The MainManager.`forcejoystick` value when it's not set to NONE starts at 0 while KeyCode joystick id starts at 1 and this isn't handled by the game correctly. This means that selecting the first connected controller actually leads to an exception being thrown when attempting to exit the settings window of the PauseMenu and setting it to any further controller actually leads to the previous one being selected

## Other InputIO controller related methods
Here are other controller related InputIO methods.

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
