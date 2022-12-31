# Settings list type

Display the list of the game settings which is platform specific.

## Options generation

`listvar` contains the indexes of the settingsindex array field. Here's a table that shows which settings is which in the exact order of the list options and under what conditions they are available:

|listvar value|settingsindex value|Description|Restrictions|
|-------------|-------------------|-----------|------------|
|0|33|Music Volume|None|
|1|34|Sound Volume|None|
|2|160|Dialogue Volume|None|
|21|261|Audio (Stero/Mono)|PC platforms only|
|19|239|Music After Battle (Resume/Restart)|None|
|20|245|Mash Action Commands|None|
|3|28|Resolution|PC platforms only|
|4|29|Fullscreen|PC platforms only|
|5|32|Downsampling\*|DISABLED ON ALL PLATFORMS|
|6|116|Anti-Aliasing|PC platforms only|
|7|30|Shadow Quality|PC platforms only|
|8|31|Texture Quality|PC platforms only|
|9|147|3D Outlines|None|
|11|156|Advanced Particles|PC platforms only|
|10|140|Wind Effects|PC platforms only|
|16|183|Vsync|PC platforms only|
|12|80|Framerate|PC platforms only, only when Vsync is OFF|
|23|255|Pause When Unfocused|PC platforms only|
|13|157|Use Controller|PC platforms only|
|17|222|Controller to Use|PC platforms only, only if Use Controller is CHOOSE MANUALLY or PRE-CONFIGURED|
|22|270|Analog Sensitivity|Only if Use Controller isn't DISABLED|
|18|231|Controller Bindings|PC platforms only, only if Use Controller is CUSTOM BINDINGS|
|14|35|Key Bindings|PC platforms only|
|15|36|Return to Main Menu / Exit Game|None on PC platforms, only available after save load on console platforms|
|24|256|Reset Control. Bindings\*\*|DISABLED ON ALL PLATFORMS|

\*: Downsampling was intended to alleviate the load of the game on the hardware by scaling down the final render by a percentage before it gets to the screen. It had the options OFF, 90%, 80%, 75%, 60%, 50% and 40% available. While it is disabled in the setting, its feature is still present and is actually used on any of the Termacade screens at 80% downsampling.

\*\*: This setting was intended to unbind every input from the rebinder, but it was deemed redundant of simply using the keyboard to remedy binding problems as well as being known to have introduced bugs during the 1.1.0 beta.

`listredirect` is overridden to null.

## Option's [SetText](../../SetText/SetText.md) input string

The text string is rendered on the left side of the bar and it corresponds to the MenuText line id of the  settingindex value of the option prepended with |[single](../../SetText/Commands/Individual%20commands/Single.md)\|. Unless otherwise specified, all option will render 2 arrows on the left and right side of the value selection that is located towards the right side of the bar. All [SetText](../../SetText/SetText.md) calls done on the right side of the bar are done in non [Dialogue mode](../../SetText/Dialogue%20mode.md) where the position is (6.25, -0.15), the bar is the parent and the input string is prepended with |[center](../../SetText/Commands/Individual%20commands/Center.md)\||[size](../../SetText/Commands/Individual%20commands/size.md),0.75|.   Here are the text and UI details per settings option:

|listvar value|settingsindex value|Description|UI rendering (right side of the bar)|
|-------------|-------------------|-----------|------------------------------------|
|0|33|Music Volume|Render a bar slider by rendering 10 sprites next to each other left to right using an index from 0 to 9. If the index is lower then pausemenu.mvolume * 10.0, the sprite is a white circle, yellow hexagon otherwise|
|1|34|Sound Volume|Render a bar slider by rendering 10 sprites next to each other left to right using an index from 0 to 9. If the index is lower then pausemenu.svolume * 10.0, the sprite is a white circle, yellow hexagon otherwise|
|2|160|Dialogue Volume|Render a bar slider by rendering 10 sprites next to each other left to right using an index from 0 to 9. If the index is lower then pausemenu.dvolume * 10.0, the sprite is a white circle, yellow hexagon otherwise|
|21|261|Audio|Calls SetText where the input string is MenuText line id 263 (STEREO) when monoaudio is false and 262 (MONO) when it is true|
|19|239|Music After Battle|Calls SetText where the input string is MenuText line id 241 (RESTART) when keepmusicafterbattle is false and 240 (RESUME) when it is true|
|20|245|Mash Action Commands|Calls SetText where the input string is MenuText line id 246 (FILL BAR) when pausemenu.mash is false and 247 (SEQUENTIAL KEYS) when it is true|
|3|28|Resolution|Calls SetText where the input string is the floored x and y component of the vector2 among the MainManager.resolution array where the index is pausemenu.resolutionid and where both component is separated by `x`|
|4|29|Fullscreen|Calls SetText where the input string is MenuText line id 39 (OFF) when pausemenu.fulls is false and 38 (ON) when it is true|
|5|32|Downsampling|Calls SetText where the input string is MenuText line id 39 (OFF) when downsample is zero. Otherwise, the text is the number in the downsamples array * 100 whose index is downsample followed by `%`|
|6|116|Anti-Aliasing|Calls SetText where the input string is MenuText line id 39 (OFF) when the FXAA component on the MainCamera is disabled and 38 (ON) when it is enabled|
|7|30|Shadow Quality|Calls SetText where the input string is MenuText line id 40 (LOW) when pausemenu.lowshadow is true and 41 (FULL) when it is false|
|8|31|Texture Quality|Calls SetText where the input string is MenuText line id 40 (LOW) when pausemenu.lowtex is true and 41 (FULL) when it is false|
|9|147|3D Outlines|Calls SetText where the input string is MenuText line id 39 (OFF) when enableoutline is 0, 40 (LOW) when it is 1 and 41 (FULL) when it is 2|
|11|156|Advanced Particles|Calls SetText where the input string is MenuText line id 39 (OFF) when particlelevel is 0, 40 (LOW) when it is 1 and 41 (FULL) when it is 2|
|10|140|Wind Effects|Calls SetText where the input string is MenuText line id 39 (OFF) when nowindeffect is true and 38 (ON) when it is false|
|16|183|Vsync|Calls SetText where the input string is MenuText line id 39 (OFF) when pausemenu.vsync is zero and 38 (ON) when it is not|
|12|80|Framerate|Calls SetText where the input string is MenuText line id 81 (30 FPS) when pausemenu.fps is 0, 82 (60 FPS) when it is 1 and 107 (Unlimited) when it is 2|
|23|255|Pause When Unfocused|Calls SetText where the input string is MenuText line id 39 (OFF) when pausemenu.pauseunfocus is false and 38 (ON) when it is true|
|13|157|Use Controller|Calls SetText where the input string depends on pausemenu.joystick\*|
|17|222|Controller to Use|If Use Controller is set to PRE-CONFIGURED, Calls SetText where the input string depends on pausemenu.joystickid\*\*. Otherwise, if pausemenu.joystickid is higher than -1, Calls SetText with \`|
|22|270|Analog Sensitivity|First, if pausemenu.analog is higher than 2, it is set to 0. Then, SetText is called where the input string is MenuText line id 39 (OFF) when pausemenu.analog is 0, 40 (LOW) when it is 1 and 41 (FULL) when it is 2|
|18|231|Controller Bindings|Does not render anything including the arrows|
|14|35|Key Bindings|Does not render anything including the arrows|
|15|36|Return to Main Menu / Exit Game\*\*\*|Does not render anything including the arrows|
|24|256|Reset Control. Bindings|Does not render anything including the arrows|

\*: Here are all the pausemenu.joystick values with their corresponding text:
joystick | MenuText line id | Description
------------ | -------- | ---------
0 | 218 | DISABLED
1 | 219 | AUTO DETECT
2 | 220 | FORCE XBOX
3 | 221 | CHOOSE MANUALLY
4 | 224 | PRE-CONFIGURED
5 | 230 | CUSTOM BINDINGS

\*\*: Here are all the pausemenu.joystickid values with their corresponding text when pausemenu.joystick is set to PRE-CONFIGURED:
joystickid | MenuText line id | Description
------------ | -------- | ---------
0 | 225 | Xbox 360
1 | 226 | Dualshock 4
2 | 227 | Switch Pro Controller
3 | 232 | Switch Joy-Con L
4 | 233 | Switch Joy-Con R
5 | 228 | Generic Controller 1
6 | 229 | Generic Controller 2
7 | 242 | Fight Pad Pro for Switch
8 | 269 | Xbox Linux

\*\*\*: An exception to the MenuText line id rule is the return to main menu option: the line id is the settingindex value only if it wasn't called from the StartMenu, but if it was, the line id is 37 instead.

## Description box rendering

It uses the default rendering scheme described in [Description box rendering](../ShowItemList%20Life%20Cycle/Description%20box%20rendering.md) where the text is MenuText line id 0 which is `berry` in English. It should be noted that under normal gameplay, this list type is not called with `showdescription` set to true.

## Input handling

The Input handling of the list is done by PauseMenu's Update instead of MainManager's.
