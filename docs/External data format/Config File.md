# Config file
There is a file persisted on disk by the game that doesn't contain any of the save data in the [save files](Save%20File.md).

This file is called `config.dat` which is persisted at the same place than the save files which is the game's directory (right next to Unity's player's .exe file). It contains other types of information to persist globally that applies regardless of the save file used. The vast majority of the data contains the settings configured.

## Config file structure
The layout of the file is similar to the save file where it contains different fields's values split by newlines. All fields contains a simple primitive value except for one which contains a list of primitives.

The amount of lines / fields in the file and which line index corresponds to which field is platform dependant meaning the exact fields and their order depends on the platform the game is built for. This page ONLY documents the format of the file the Steam, GOG and Itch.io versions of the game on PC.

The parsing of the file is done by MainManager.ReadSettings method:

```cs
public static void ReadSettings(string[] c)
```
It sets all fields backed by a setting value where `c` is the string array read by reading the `config.dat` file.

To get the string to save to the file MainManager.SaveSettings is called:

```cs
public static string SaveSettings()
```
It returns a `config.dat` compliant text from the current settings fields's values, the same ones that were read by ReadSettings, but it takes into account the changes that happened during the game session.

## `Config.dat` fields table
The following table shows the fields present in `config.dat` and their format ordered by their line index in the file:

|Line|Format|Name|Description|
|---:|------|----|----------|
|0|A KeyCode enum value as string or int|InputIO.`keys[0]`|The keybaord input bound to the up game input|
|1|A KeyCode enum value as string or int|InputIO.`keys[1]`|The keybaord input bound to the down game input|
|2|A KeyCode enum value as string or int|InputIO.`keys[2]`|The keybaord input bound to the left game input|
|3|A KeyCode enum value as string or int|InputIO.`keys[3]`|The keybaord input bound to the right game input|
|4|A KeyCode enum value as string or int|InputIO.`keys[4]`|The keybaord input bound to the confirm game input|
|5|A KeyCode enum value as string or int|InputIO.`keys[5]`|The keybaord input bound to the cancel game input|
|6|A KeyCode enum value as string or int|InputIO.`keys[6]`|The keybaord input bound to the switch party game input|
|7|A KeyCode enum value as string or int|InputIO.`keys[7]`|The keybaord input bound to the toggle HUD game input|
|8|A KeyCode enum value as string or int|InputIO.`keys[8]`|The keybaord input bound to the pause game input|
|9|A KeyCode enum value as string or int|InputIO.`keys[9]`|The keybaord input bound to the help game input|
|10|int between 0 and 7|MainManager.`resolutionindex`|The game resolution to use among different presets. Here are the mapping of the values to their resolutions:<ol start="0"><li>1024 x 576</li><li>1152 x 648</li><li>1280 x 720</li><li>1366 x 768</li><li>1600 x 900</li><li>1920 x 1080</li><li>2560 x 1440</li><li>3840 x 2160</li></ol>|
|11|`True` or `False`|MainManager.`fullscreen`|Tells whether or not the game should be rendered in fullscreen mode. If it isn't, it is rendered in windowed mode|
|12|int between 0 and 2|MainManager.`fps`|Tells what value shouls be set to Application.targetFrameRate:<ol start="0"><li>30</li><li>60 (the default)</li><li>-1 (unlimited)</li></ol> NOTE: The value 2 is not selectable under normal gameplay, but will still cause the game to be rendered without an FPS limit if it is read from the file. Doing so is NOT recommended because most of the game's logic isn't designed to go beyond 60 FPS|
|13|`True` or `False`|MainManager.`lowshadows`|Whether or not QualitySettings.shadowResolution should be set to Low. If the value is false, it is set to High|
|14|`True` or `False`|MainManager.`lowtexture`|Whether or not QualitySettings.masterTextureLimit should be set to 1. If the value is false, it is set to 0|
|15|float between 0.0 and 1.0|MainManager.`musicvolume`|The base value to scale [music playback](../General%20systems/Music%20playback.md)'s volume|
|16|float between 0.0 and 1.0|MainManager.`soundvolume`|The base value to scale [sound playback](../General%20systems/Sounds%20playback.md)'s volume|
|17|`True` or `False`|The enablement of the FXAA compoent of the `MainCamera`|Tells if the FXAA shader should apply when rendering the game|
|18|int corresponding to a valid [languageid](../SetText/languageid.md)|MainManager.`languageid`|The language the game is set to. This is the only required value for the game to [boot](../MainManager/Boot%20and%20reset%20process.md) to the StartMenu and it influences a lot of logic in the game that are language dependant or changes several assets being read|
|19|`True` or `False`|MainManager.`nowindeffect`|Tells whether or not any wind effects should be disabled. They are left enabled if the value is false. More specifically, it causes any Wind component to leave the GameObject attached disabled (done on the Wind's Start) and it prevents any [BeetleGrass](../Entities/NPCControl/ObjectTypes/BeetleGrass.md) from getting the MainManager.`windShader` on their SetUp|
|20|int between 0 and 2|MainManager.`enableoutline`|Determines which value to set the `GlobalOutline` global shader property on MainManager.UpdateOutlines:<ol start="0"><li>0.0</li><li>0.1</li><li>0.3 (Doesn't apply if the current [area](../Enums%20and%20IDs/librarystuff/Areas.md) is `MetalLake` where the value will be 0.1 instead)</li></ol> NOTE: The value is forced to 1 temporarily during event 204 (credits) regardless of the original value, but it is restored towards the end of the event|
|21|int between 0 and 6|MainManager.`downsample`|Determines the factor at which to downsample the render texture of the camera. Here are the mappings of the values:<ol start="0"><li>1.0</li><li>0.9</li><li>0.8</li><li>0.75</li><li>0.6</li><li>0.5</li><li>0.4</li></ol>NOTE: This setting is not configurable under normal gameplay and the value from the config file is ignored|
|22|int between 0 and 2|MainManager.`particlelevel`|Controls the rendering of different particles effects. Here are the behaviors of each values:<ol start="0"><li>All LilypadEffects gets disabled on their Start, all GameObject with a tag of `Particable` gets disabled on the first LateUpdate of MapControl and all GameObjects attached to a PrefabParticle gets disabled (done on the component's Start)</li><li>None of the effects described above applies, but every PrefabParticle component has their `maxammount` halved and floored on their Start</li><li>None of the effects described in the above values applies</li></ol>|
|23|int between 0 and 5|MainManager.`usejoystick`|Tells how the game should treat controllers, here are the values mapping:<ol start="0"><li>DISABLED</li><li>AUTO DETECT</li><li>FORCE XBOX</li><li>CHOOSE MANUALLY</li><li>PRE-CONFIGURED</li><li>CUSTOM BINDINGS</li></ol>|
|24|float between 0.0 and 1.0|MainManager.`bleepvolume`|The base value to scale [bleeps](../SetText/bleep.md)'s volume|
|25|int|MainManager.`vsync`|Determines the value of QualitySettings.vSyncCount that the game will use. If the config file value is 0, the vSyncCount used will be 0 (vsync is disabled). Otherwise (any other value than 0), vSyncCount will be set to Screen.currentResolution.refreshRate / 60.0 floored and clamped from 1 to 4 (vsync is enabled, but in a way to target the closest FPS number to 60.0 such that the FPS is still at least 60.0)|
|26|int being -1 or a valid controller id TODO: detail|MainManager.`forcejoystick`|<ol start="0"><li>Xbox 360</li><li>Dualshock 4</li><li>Switch Pro Controller</li><li>Switch Joy-Con L</li><li>Switch Joy-Con R</li><li>Generic Controller 1</li><li>Generic Controller 2</li><li>Fight Pad Pro for Switch</li><li>Xbox Linux</li></ol>|
|27|`True` or `False`|MainManager.`keepmusicafterbattle`|Determines if the music timestamp should be saved when entering a battle and restored when leaving it. NOTE: Some maps override this logic and allows an [incorrect playback issue](../General%20systems/Music%20playback.md#issue-with-musicresume) to occur under specific conditions when this setting is enabled|
|28|`True` or `False`|MainManager.`mashcommandalt`|If true, some [ActionCommands](../Battle%20system/ActionCommands.md) will have their command overriden to another|
|29|int|MainManager.`joybinds[0]`|The controller input bound to the up game input|
|30|int|MainManager.`joybinds[1]`|The controller input bound to the down game input|
|31|int|MainManager.`joybinds[2]`|The controller input bound to the left party game input|
|32|int|MainManager.`joybinds[3]`|The controller input bound to the right game input|
|33|int|MainManager.`joybinds[4]`|The controller input bound to the confirm game input|
|34|int|MainManager.`joybinds[5]`|The controller input bound to the cancel game input|
|35|int|MainManager.`joybinds[6]`|The controller input bound to the switch party game input|
|36|int|MainManager.`joybinds[7]`|The controller input bound to the toggle HUD game input|
|37|int|MainManager.`joybinds[8]`|The controller input bound to the pause game input|
|38|int|MainManager.`joybinds[9]`|The controller input bound to the help game input|
|39|`True` or `False`|MainManager.`monoaudio`|Determines if AudioSettings.speakerMode should be set to `Mono`. If the value is false, it is set to `Stereo`|
|40|A `,` seprated list of bool, each containing `True` or `False`|MainManager.`secretunlocks`|The list of secret codes allows to be toggled on the StartMenu's secret menu when creating a new file (it can be accessed by pressing the Help input when selecting an empty save slot). The list is normally 5 elements with the indexes mapping to the following codes:<ol start="0"><li>RUIGEE</li><li>HARDEST</li><li>FRAMEONE</li><li>MOREFARM</li><li>MYSTERY?</li></ol>NOTE: At least 2 elements in the list must be true for the secret menu to be accessible|
|41|int between 0 and 2|MainManager.`analog`|Controls the sensitivity of analogue input when moving in the overworld. It determines the value of [walkdelta](../PlayerControl/Update.md#walkdelta-computation) on PlayerControl.Update and controls some [Movement](../Entities/EntityControl/Notable%20methods/Follow.md#movement-logic) on PlayerControl.Movement. 0 means OFF, 1 means LOW and 2 means HIGH|
|42|`True` or `False`|MainManager.`pauseonfocus`|Determines the value of Application.runInBackground. If the config file value is false, Application.runInBackground is set to true. If the config file value is true, Application.runInBackground is set to false|
|43|`True` or `False`|MainManager.`snapTo8`|If true, it changes the logic of [DoActionTap](../PlayerControl/Actions/DoActionTap.md) when `Bee` is the party leader where the Beemerang's heading direction will be the player's heading direction snapped to a vector whose components will be snapped to -1.0, 0.0 or 1.0 depending on the closest one (so 8 cardinal directions)|

> NOTE: Any fields with an index of 29 and above are optional, but if any is omited, field 29 and any further field won't be read from the file. Also, field 26 (MainManager.`forcejoystick`) is optional if field 23 (MainManager.`usejoystick`) isn't 4 or 5 (it's not PRE-CONFIGURED or CUSTOM BINDINGS), but if field 26 is present, field 27 and 28 become required.

## ApplySettings
There is another method that applies all changes done to settings values during the game without necessarily saving them directly. That method is MainManager.ApplySettings:

```cs
public static void ApplySettings()
```
Most of the logic this method does is explained in the table above, but there's also some additional logic. Here are the additional steps the method does:

- If `music[0]` is currently playing while instance.`inmusicrange` is -1 (no [MusicRange](../Entities/NPCControl/ObjectTypes/MusicRange.md) is active), `music[0]`.volume is set to the new `musicvolume` value
- All `sounds`'s volume are set to the new `soundvolume` value
- If `usejoystick` is 5 (CUSTOM BINDINGS), `joyid` is set to the new `forcejoystick` value
- If `usejoystick` is 3 (CHOOSE MANUALLY) while `forcejoystick` isn't negative TODO: ??? or if `usejoystick` is 0 (DISABLED) while `joystick` is true TODO: ???:
    - InputIO.GetJoyButtons called
    - `forcecontrollerupdate` set to true
- If AudioSettings.speakerMode indicates a different value than the new `monoaudio` value and `music[0]` is currently playing an AudioClip, it is replayed by doing the following:
    - [ChangeMusic](../General%20systems/Music%20playback.md#changemusic) called with `music[0]`.clip.name
    - `music[0]`.Play called
    - `music[0]`.time is reset to the value it had before the above happened
- If InputIO.`isConsole` is true (a console platform), Application.runInBackground is set to false
- If Application.isMobilePlatform is false, the resolution settings are ignored and the Screen resolution is instead set to 1024 x 576 without fullscreen
- If `map` exists (a map isn't being loaded), `map`.RefreshSoundVolume is called which sets all SoundControl's `source`.volume present in the scene to their `startvalue` * the new `soundvolume` value
- If `analog` is higher than 2, it is set to 0. This can't happen under normal gameplay
